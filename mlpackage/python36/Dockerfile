FROM nvidia/cuda:9.0-cudnn7-runtime


RUN apt-get update && \
	apt-get install -y \
	software-properties-common \
	python-software-properties && \
	add-apt-repository ppa:jonathonf/python-3.6 && \
	apt-get update && \
	apt-get install -y python3.6
	
RUN cd /usr/bin && \
	ln -sfn python3.6 python3 && \
	ln -sfn python3 python && \
	ln -sfn python3.6m python3m && \
	ln -sfn pydoc3.6 pydoc3 && \
	ln -sfn pydoc3 pydoc && \
	ln -sfn pdb3.6 pdb3 && \
	ln -sfn pdb3 pdb && \
	ln -sfn pygettext3.6 pygettext3 && \
	ln -sfn pygettext3 pygettext

RUN apt-get install -y curl

RUN curl https://bootstrap.pypa.io/get-pip.py -o get-pip.py && \
	python get-pip.py

COPY requirements.txt /tmp/requirements.txt
RUN pip install -r /tmp/requirements.txt

WORKDIR /notebooks

EXPOSE 8888

RUN python -m ipykernel.kernelspec
CMD ["bash", "-c", "source /etc/bash.bashrc && jupyter notebook --notebook-dir=/notebooks --ip 0.0.0.0 --no-browser --allow-root"]
