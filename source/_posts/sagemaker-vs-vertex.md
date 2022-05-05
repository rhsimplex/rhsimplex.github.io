---
title: SageMaker vs Vertex in early 2022
date: 2022-05-05 15:38:43
tags: [mlops, aws, gcp]
---


To ensure, ahem, *efficient* usage of {% post_link cloud-credit-where-credit-is-due 'startup cloud credits' %}, I've recently completed a machine learning platform migration from Google's [VertexAI](https://cloud.google.com/vertex-ai), to Amazon's [SageMaker](https://docs.aws.amazon.com/sagemaker/latest/dg/whatis.html). I want to share my limited impressions about each platform.

## Which is better? 

VertexAI is clunkier to use, but feels more "correct" in a software-engineering sense. SageMaker is more straight-forward for a technical user without much software engineering background -- the archetypical data scientist -- but has some anti-patterns that make integration a bit more treacherous.

## Context

I did the migration in early 2022.   My use case is training neural networks, and in particular hyperparameter optimization. I have a CI pipeline (outside of AWS or GCP) that builds and tests the model code and pushes a container image to the platforms' respective container registries. Most of the model code is PyTorch.

I don't want to pretend to have covered everything these platforms offer, which is a lot. To summarize I have, a bit of experience on each platform with:

* Containerized Training 
* Hyperparameter searches
* Jupyter notebooks
* S3/GCS integration

And a non-exhaustive list of things I haven't used yet:
* Model deployment/lifecycle
* Data pipelines
* Auto ML

## What they have in common

I haven't worked with cloud ML platforms in a few years, and I'm happy to say they've come a long way. Both SageMaker and VertexAI do what I expected with only minor headaches. Both apply a cost markup to compute resources used for training (e.g. the training instance used on SageMaker will cost more per hour than the same instance on EC2), and unfortunately it's not at clear what percentage this is or if it's stable over time. VertexAI has a slight edge here because you can choose your accelerator, i.e. GPU, separately from the base instance. In SageMaker, they're lumped together.

One surprising annoyance is the UI for either can be very slow--surprisingly, frustratingly slow. I try to use the CLI or write my own scripts with the respective SDKs, but I would expect one of the advantages of using a platform like SageMaker or VertexAI would be the slick, informative UI. Sadly, this isn't the case. 

## How much will I need to change my training code?

In an ideal world, you should not have to modify your training code at all to integrate it into a machine learning platform. This is not the case with SageMaker or VertexAI, but SageMaker is a bit worse in this respect. 

If you've written and containerized a training script, you probably have a Dockerfile that ends with something like:

```dockerfile
ENTRYPOINT ["python", "train.py"]
```

That way, arguments (hyperparameters) you pass to the container are passed to the script directly. VertexAI does this by default. I've since learned that Sagemaker [can also do this](https://github.com/aws/sagemaker-training-toolkit), but that wasn't initially obvious to me from the sprawling documentation. By default, Sagemaker drops a `hyperparameters.json` into your container while sending its own arguments to the container. So if you don't want to add another dependency to your build, you might have to add something like this before your `train.py` script:


```python
if __name__ == '__main__':
    if sys.argv[1] == 'train':
        # You are training withing Amazon Sagemaker, which inserts this `train` argument at the beginning.
        # You'll need to extract hyperparameters from a JSON file. This is a hack, but Sagemaker doesn't
        # provide a way to pass arguments to the container, as far as I know.
        extracted_command_line_args = []
        with open('/opt/ml/input/config/hyperparameters.json', 'r') as read_file:
            parameters = json.load(read_file)
            for key, value in parameters.items():
                if key == '_tuning_objective_metric':
                    # Another parameter inserted by SageMaker which can be ignored
                    continue
                if value == 'NULL':
                    # Some parameters don't take an argument. Sagemaker can't really handle this.
                    # so you could pass it into AWS environments like --toggle_something=NULL. The 'NULL' will be dropped.
                    extracted_command_line_args.append(key)
                    continue
                extracted_command_line_args.append(f'{key}={value}')
        args = parse_args(sys.argv[2:] + extracted_command_line_args)
    else:
        args = parse_args(sys.argv[1:])
    train(args)
```

Another pitfall this example doesn't show is that while the Python `ArgumentParser` is capable of handling repeated arguments to make a sequence (for example: `--dataset dogs --dataset cats --dataset raccoons` could give you a `dataset: [dogs, cats, raccoons]`), since SageMaker send hyperparameters around in key value pairs in a JSON (or through a `dict` if you're using the Python SDK), you can't use this pattern.

Both platforms will mount a storage volume (S3 or GCS bucket) with your training data. So here you can either supply your training script with these paths or if you're lazy like me, just copy everything to the directory you expect it to be.

```python
if os.path.isdir('/gcs/your-bucket-name/'):
    # you're on Vertex 
    ...
elif os.path.isdir('/opt/ml/input/data/'):
    # you're on SageMaker 
    ...
```

Similarly, both platform provide a mounted directory to send model artifacts to (like checkpoints and logs). Both S3 and GCS have nice `rsync`-like functionality so you can download what you need after training. SageMaker compresses everything in your artifact directory before storing it. This sounds like a good idea, but is very annoying if you just want to download, say, the Tensorboard logs without a massive model checkpoint. SageMaker also doesn't allow you to SSH into a running training container as far as I can tell, and VertexAI does (but it's not enabled by default).

## Hyperparameter tuning

If you're tuning hyperparameters, you need some way to send validation results from individual runs back to the tuner. SageMaker handles this elegantly by letting you provide a regular expression that it uses to parse the logs. This means you don't need to change your training code at all as long as you're logging the value you want to tune. Vertex requires your training code to send values to the tuner directly, for example with its [cloudlml-hypertune](https://github.com/GoogleCloudPlatform/cloudml-hypertune) package. This isn't a big deal, but it feels unnecessary to include another dependency for this.

The Vertex HPS supports [conditional hyperparemeters](https://cloud.google.com/vertex-ai/docs/training/hyperparameter-tuning-overview) and SageMaker, as far as I can see, does not. A conditional hyperparameter just means the hyperparameter will only be tuned if some condition is met. For instance, you may want to tune models on an L2 or smooth L1 loss, and in the latter case you would want to also tune the alpha parameter. In SageMaker this is not possible, and the alpha parameter will be "tuned" even when using L2 loss.

Neither supports [pruning jobs](https://optuna.readthedocs.io/en/v1.0.0/tutorial/pruning.html) that I can see, which is a shame because it could potentially save you a lot of money.

## Tensorboard integration

Vertex AI has a nice tensorboard integration that is [kind of a pain](https://cloud.google.com/vertex-ai/docs/experiments/tensorboard-training) to set up but works pretty well once it's running. Just make sure you configure the mysterious service account correctly. Oh, and I hope you're ok with the [**absurd $300 per user per month cost**](https://cloud.google.com/vertex-ai/pricing#tensorboard).

Even though Tensorboard can generate PR curves, the VertexAI version mysteriously doesn't support them (as of January 2022). Of course, I'm only assuming that's the case since my PR-curves render correctly when serving Tensorboard locally. I couldn't find any information about which version of Tensorboard the VertexAI platform runs or any differences between the VertexAI vs. vanilla Tensorboard. Considering Tensorboard is created by Google, this is pretty puzzling.

SageMaker [has a Tensorboard integration as well](https://sagemaker.readthedocs.io/en/stable/amazon_sagemaker_debugger.html?highlight=tensorboard#capture-real-time-tensorboard-data-from-the-debugging-hook), but I have not managed to get this working yet. I will update this section if I ever do. Tensorboard can take an [S3 URI directly](https://stackoverflow.com/questions/47425882/tensorboard-logdir-with-s3-path) which is pretty cool, but as I mentioned above, SageMaker is compressing your model artifacts, so this doesn't work out of the box.

## Notebooks

Both platforms use a customized JupyterLab setup for notebooks, so you will hopefully be on familiar territory here. Notably, the VertexAI instances seem to persist the conda environments between sessions, while SageMaker resets them. Both are valid design choices in my opinion.  If you want to run some setup code for SageMaker to recreate your environment, you can [easily do so](https://docs.aws.amazon.com/sagemaker/latest/dg/notebook-lifecycle-config.html).

However, I couldn't find an elegant way to integrate an external Git repository in either case. I ended up adding an SSH key to each notebook instance on which I wanted to pull code with Git.

## GUI

This will largely be a matter of taste, but the AWS UI is in general more polished and logical than GCP. Browsing logs in particular seems weirdly complicated on GCP. Also filtering (e.g. training jobs, of which you may have thousands) in Vertex only works from the beginning of the name while the SageMaker does a full-text search. As I said above, both GUIs are pretty uninspired.

## Documentation

The platform-level docs for both platforms are a bit overwhelming but complete. If anything SageMaker has too much documentation to search through effectively. There are probably [thousands of Jupyter notebooks](https://github.com/aws/amazon-sagemaker-examples) that show you how to do everything you could possibly want. I personally don't like having to search through these as documentation, but I appreciate the commitment to providing working examples for nearly every functionality. 

The SDK documentation [is a lot better for SageMaker](https://sagemaker.readthedocs.io/en/stable/); someone actually cared to organize this and make it readable. The [VertexAI counterpart](https://googleapis.dev/python/aiplatform/latest/aiplatform.html) is probably autogenerated and will require some work to get your bearings.

## Conclusion

Both VertexAI and SageMaker are much more than MLOps platforms, which can be a blessing for some but a curse for others. They seem to be trying to cover every ML use-case and every kind of user, which can add friction if you have a very specific idea of what you want to accomplish.