===============================
SageMaker TensorFlow Containers
===============================

SageMaker TensorFlow Containers is an open source library for making the
TensorFlow framework run on `Amazon SageMaker <https://aws.amazon.com/documentation/sagemaker/>`__.

This repository also contains Dockerfiles which install this library, TensorFlow, and dependencies
for building SageMaker TensorFlow images.

For information on running TensorFlow jobs on SageMaker: `Python
SDK <https://github.com/aws/sagemaker-python-sdk>`__.

For notebook examples: `SageMaker Notebook
Examples <https://github.com/awslabs/amazon-sagemaker-examples>`__.

-----------------
Table of Contents
-----------------
.. contents::
    :local:

Getting Started
---------------

Prerequisites
~~~~~~~~~~~~~

Make sure you have installed all of the following prerequisites on your
development machine:

- `Docker <https://www.docker.com/>`__

For Testing on GPU
^^^^^^^^^^^^^^^^^^

-  `Nvidia-Docker <https://github.com/NVIDIA/nvidia-docker>`__

Recommended
^^^^^^^^^^^

-  A Python environment management tool. (e.g.
   `PyEnv <https://github.com/pyenv/pyenv>`__,
   `VirtualEnv <https://virtualenv.pypa.io/en/stable/>`__)

Building your Image
-------------------

`Amazon SageMaker <https://aws.amazon.com/documentation/sagemaker/>`__
utilizes Docker containers to run all training jobs & inference endpoints.

The Docker images are built from the Dockerfiles specified in
`Docker/ <https://github.com/aws/sagemaker-tensorflow-containers/tree/master/docker>`__.

The Docker files are grouped based on TensorFlow version and separated
based on Python version and processor type.

The Docker images, used to run training & inference jobs, are built from
the corresponding "final" Dockerfiles.

Final Images
~~~~~~~~~~~~

The "final" Dockerfiles encompass the installation of the SageMaker specific support code.

Then prepare the SageMaker TensorFlow Container python package in the image folder like below:

::

    python setup.py sdist

    #. Copy your Python package to “final” Dockerfile directory that you are building.
    cp dist/sagemaker_tensorflow_container-1.0.0.tar.gz docker/1.11.0/final/py3

Then run:

::

    # All build instructions assumes you're building from the same directory as the Dockerfile.
    cd docker/1.11.0/final/py3/

    # GPU
    docker build -t elgalu/tf-1.11.0-gpu-py3:0 -f Dockerfile.gpu .

* For information about downloading the enhanced versions of TensorFlow serving, see `Using TensorFlow Models with Amazon EI <https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ei-tensorflow.html>`__.

Push to your ECR Repo
---------------------

::

    AWS_ACCOUNT_NUMBER=012345678901
    $(aws ecr get-login --profile=default --region=eu-central-1 --registry-ids=${AWS_ACCOUNT_NUMBER} --no-include-email)
    aws ecr create-repository --profile=default --region=eu-central-1 --repository-name sagemaker-tensorflow-reco:1.11-gpu-py3
    docker push "${AWS_ACCOUNT_NUMBER}.dkr.ecr.eu-central-1.amazonaws.com/sagemaker-tensorflow-reco:1.11-gpu-py3"


Running the tests
-----------------

Running the tests requires installation of the SageMaker TensorFlow Container code and its test
dependencies.

::

    git clone https://github.com/aws/sagemaker-tensorflow-containers.git
    cd sagemaker-tensorflow-containers
    pip install -e .[test]

Tests are defined in
`test/ <https://github.com/aws/sagemaker-tensorflow-containers/tree/master/test>`__
and include unit, integration and functional tests.

Unit Tests
~~~~~~~~~~

If you want to run unit tests, then use:

::

    # All test instructions should be run from the top level directory

    pytest test/unit

Integration Tests
~~~~~~~~~~~~~~~~~

Running integration tests require `Docker <https://www.docker.com/>`__ and `AWS
credentials <https://docs.aws.amazon.com/sdk-for-java/v1/developer-guide/setup-credentials.html>`__,
as the integration tests make calls to a couple AWS services. The integration and functional
tests require configurations specified within their respective
`conftest.py <https://github.com/aws/sagemaker-tensorflow-containers/blob/master/test/integ/conftest.py>`__.

Integration tests on GPU require `Nvidia-Docker <https://github.com/NVIDIA/nvidia-docker>`__.

Before running integration tests:

#. Build your Docker image.
#. Pass in the correct pytest arguments to run tests against your Docker image.

If you want to run local integration tests, then use:

::

    # Required arguments for integration tests are found in test/integ/conftest.py

    pytest test/integ --docker-base-name <your_docker_image> \
                      --tag <your_docker_image_tag> \
                      --framework-version 1.11.0 \
                      --processor <cpu_or_gpu>

::

    # Example
    pytest test/integ --docker-base-name preprod-tensorflow \
                      --tag 1.0 \
                      --framework-version 1.4.1 \
                      --processor cpu

Functional Tests
~~~~~~~~~~~~~~~~

Functional tests require your Docker image to be within an `Amazon ECR repository <https://docs
.aws.amazon.com/AmazonECS/latest/developerguide/ECS_Console_Repositories.html>`__.

The `docker-base-name` is your `ECR repository namespace <https://docs.aws.amazon
.com/AmazonECR/latest/userguide/Repositories.html>`__.

The `instance-type` is your specified `Amazon SageMaker Instance Type
<https://aws.amazon.com/sagemaker/pricing/instance-types/>`__ that the functional test will run on.


Before running functional tests:

#. Build your Docker image.
#. Push the image to your ECR repository.
#. Pass in the correct pytest arguments to run tests on SageMaker against the image within your ECR repository.

If you want to run a functional end to end test on `Amazon
SageMaker <https://aws.amazon.com/sagemaker/>`__, then use:

::

    # Required arguments for integration tests are found in test/functional/conftest.py
    pytest test/functional --aws-id <your_aws_id> \
                           --docker-base-name <your_docker_image> \
                           --instance-type <amazon_sagemaker_instance_type> \
                           --tag <your_docker_image_tag> \

::

    # Example
    pytest test/functional --aws-id 12345678910 \
                           --docker-base-name preprod-tensorflow \
                           --instance-type ml.m4.xlarge \
                           --tag 1.0

If you want to run a functional end to end test for your Elastic Inference container, you will need to provide an `accelerator_type` as an additional pytest argument.

The `accelerator-type` is your specified `Amazon Elastic Inference Accelerator <https://aws.amazon.com/sagemaker/pricing/instance-types/>`__ type that will be attached to your instance type.

::

    # Example for running Elastic Inference functional test
    pytest test/functional/test_elastic_inference.py --aws-id 12345678910 \
                                                     --docker-base-name preprod-tensorflow \
                                                     --instance-type ml.m4.xlarge \
                                                     --accelerator-type ml.eia1.medium \
                                                     --tag 1.0

Contributing
------------

Please read
`CONTRIBUTING.md <https://github.com/aws/sagemaker-tensorflow-containers/blob/master/CONTRIBUTING.md>`__
for details on our code of conduct, and the process for submitting pull
requests to us.

License
-------

SageMaker TensorFlow Containers is licensed under the Apache 2.0 License. It is copyright 2018
Amazon.com, Inc. or its affiliates. All Rights Reserved. The license is available at:
http://aws.amazon.com/apache2.0/
