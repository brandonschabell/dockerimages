FROM nvidia/cuda:9.0-cudnn7-runtime


RUN apt-get update && \
	apt-get install -y \
	software-properties-common \
	python-software-properties && \
	add-apt-repository ppa:jonathonf/python-3.6 && \
	apt-get update && \
	apt-get install -y python3.6

COPY requirements.txt /tmp/requirements.txt
RUN pip install -r /tmp/requirements.txt

WORKDIR /notebooks

EXPOSE 8888

RUN ${PYTHON} -m ipykernel.kernelspec
CMD ["bash", "-c", "source /etc/bash.bashrc && jupyter notebook --notebook-dir=/notebooks --ip 0.0.0.0 --no-browser --allow-root"]
