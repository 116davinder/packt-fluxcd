FROM python:3.12-alpine3.19 

RUN mkdir -p /app 

WORKDIR /app 

COPY requirements.txt /app 

RUN pip install -r requirements.txt 

EXPOSE 5000 

COPY . .

CMD ["python", "app.py"]