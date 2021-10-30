FROM python:3.8-alpine3.13
ADD . /api
WORKDIR /api
RUN pip install -r requirements.txt
EXPOSE 80
CMD ["python", "src/main.py"]