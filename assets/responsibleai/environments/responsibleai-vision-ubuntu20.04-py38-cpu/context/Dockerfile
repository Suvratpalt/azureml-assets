FROM mcr.microsoft.com/azureml/openmpi4.1.0-ubuntu20.04:{{latest-image-tag}}
# Install OpenCV native dependencies
RUN apt-get update
RUN apt-get install -y python3-opencv
RUN apt-get remove -y libavutil56 libswresample3 libavformat58 libavcodec58 libswscale5 libopenexr24

ENV AZUREML_CONDA_ENVIRONMENT_PATH /azureml-envs/responsibleai-vision

# Prepend path to AzureML conda environment
ENV PATH $AZUREML_CONDA_ENVIRONMENT_PATH/bin:$PATH

# Create conda environment
COPY conda_dependencies.yaml .
RUN conda env create -p $AZUREML_CONDA_ENVIRONMENT_PATH -f conda_dependencies.yaml -q && \
    rm conda_dependencies.yaml && \
    conda run -p $AZUREML_CONDA_ENVIRONMENT_PATH pip cache purge && \
    conda clean -a -y

RUN pip install --pre 'azure-ai-ml' 'azure-storage-blob<=12.13.0'

# Install azureml packages
RUN pip install 'azureml-core=={{latest-pypi-version}}' \
                'azureml-mlflow=={{latest-pypi-version}}' \
                'azureml-rai-utils==0.0.6'

RUN pip install 'azureml-dataset-runtime=={{latest-pypi-version}}' \
                'azureml-automl-dnn-vision=={{latest-pypi-version}}'

RUN pip install 'shap==0.41.0' \
                'interpret-community==0.30.0' \
                'Pillow>=9.1.0' \
                'Werkzeug==2.2.3' \
                'setuptools>=65.5.1' \
                'numpy==1.22.0' \
                'scipy==1.10.0' \
                'statsmodels==0.14.0' \
                'torch==1.13.1'

RUN pip freeze

# This is needed for mpi to locate libpython
ENV LD_LIBRARY_PATH $AZUREML_CONDA_ENVIRONMENT_PATH/lib:$LD_LIBRARY_PATH
