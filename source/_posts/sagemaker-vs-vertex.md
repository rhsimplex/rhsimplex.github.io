---
title: Sagemaker vs Vertex in early 2022
date: 2022-04-27 15:38:43
tags: [mlops, aws, gcp]
---

To ensure, ahem, *efficient* usage of [startup cloud credits](2022/04/26/cloud-credit-where-credit-is-due/index.html), I've recently completed a migration from GCP's MLOps platform [VertexAI](https://cloud.google.com/vertex-ai), to AWS's [SageMaker](https://docs.aws.amazon.com/sagemaker/latest/dg/whatis.html). I want to share with you my limited impressions of and gripes about each platform.

## Which is better? 

VertexAI is clunkier to use, but feels more "correct" in a software-engineering sense. SageMaker is more straight-forward for a technical user without much software engineering background -- the archetypical data scientist -- but has some anti-patterns that make integration irritating.

## Context

I did the migration in early 2022.   My use case is so far managing neural network training, in particular hyperparameter optimization. I have a CI pipeline that builds and tests the model code and pushes a container image to the platforms' respective container registries. Most of the model code is PyTorch.

I don't want to pretend to have covered everything these platforms offer. To summarize:

Have experience:
* Training with container images
* Hyperparameter searches
* Notebooks
* S3/GCS integration

Didn't try yet:
* Model deployment/lifecycle
* Data pipelines
* Auto ML

## What they have in common

I haven't worked with cloud ML platforms in a few years, and I'm happy to say they've come a long way. Both SageMaker and VertexAI do what I expected with only minor headaches.Both apply a markup to compute resources used for training (e.g. the training instance used on SageMaker will cost more per hour than the same instance on EC2), and naturally it's not at all clear what percentage this is or if it's stable over time.

One surprising annoyance is the UI for either can be very slow. I try to use the CLI or write my on scripts with the respective SDKs, but I would expect one of the advantages of using a MLOps platform 

## How much will I need to change my training code?

Ideally, if you structure your code cleanly, you should not have to modify it at all to integrate into a machine learning platform. Unfortunately, this is not the case with SageMaker or VertexAI, but SageMaker is much worse in this respect. 

If you've written and containerized a training script, you probably have a Dockerfile that ends with something like:

```dockerfile
ENTRYPOINT ["python", "train.py"]
```

That way, arguments (hyperparameters) you pass to the container are passed to the script directly. VertexAI does this -- correctly, in my opinion -- and SageMaker does not. Sagemaker instead drops a `hyperparameters.json` into your container while sending its own arguments to the container. So you might have to add something like this before your `train.py` script: 

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

Another pitfall this example doesn't show is that while the Python `ArgumentParser` is capable of handling repeated arguments to make a sequence (for example: `--parameter 1 --parameter 2 --parameter 3` could give you a `parameter: [1, 2, 3]`), since SageMaker is compressing them in to key value pairs in a JSON, you can't use this pattern. Naturally, it silently takes the last one.

Both platforms will mount a storage volume (S3 or GCS bucket) with your training data. So here you can either supply your training script with these paths or if you're lazy like me, just copy everything to the directory you expect it to be.

```python
if os.path.isdir('/gcs/your-bucket-name/'):
    # you're on Vertex 
    ...
elif os.path.isdir('/opt/ml/input/data/'):
    # you're on SageMaker 
    ...
```

Similarly, both platform provide a mounted directory to send model artifacts to (like checkpoints and logs). Both S3 and GCS have nice `rsync`-like functionality so you can download what you need after training. SageMaker compresses everything in your artifact directory before storing it. This sounds like a good idea, but is very annoying if you just want to download, say, the Tensorboard logs without a massive model checkpoint.


## Tensorboard integration

Vertex AI has a nice tensorboard integration that is [kind of a pain](https://cloud.google.com/vertex-ai/docs/experiments/tensorboard-training) to set up but works pretty well once it's running. Just make sure you configure the mysterious service account correctly. Oh, and I hope you're ok with the [**absurd $300 per user per month cost**](https://cloud.google.com/vertex-ai/pricing#tensorboard). Ever buy a Coke in a strip club only to find out it cost $300 when you try to leave? No reason.

Anyway, hope you don't need precision-recall curves. Even though Tensorboard can generate PR curves, the VertexAI version mysteriously doesn't support them (as of January 2022). Of course, I'm only assuming that's the case since my PR-curves render correctly when serving Tensorboard locally. I couldn't find any information about which version of Tensorboard the VertexAI platform runs or any differences between the VertexAI vs. vanilla Tensorboard. Considering Tensorboard is created by Google, this is pretty puzzling.

## Vertex AI
* container args
* bad filtering
* irritating logging
* $300 per user for tensorboard!

## sagemaker
* no container args, config.yml, injects own arguments
* no complex hps parameters
* accelerator and instances combined
* couldn't get tensorboard integration working
* notebooks reset environments
* zips everything