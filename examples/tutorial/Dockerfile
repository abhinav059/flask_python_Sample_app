FROM python:3.11-slim

WORKDIR /app

COPY . .

RUN pip install --upgrade pip

RUN pip install .

EXPOSE 5000

CMD sh -c "flask --app flaskr init-db && flask --app flaskr run --host=0.0.0.0"