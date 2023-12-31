FROM alpine:latest

RUN apk update && apk add postgresql postgresql-contrib postgresql-dev gcc python3-dev musl-dev

RUN python3 -m ensurepip && pip3 install --upgrade pip

COPY requirements.txt /pr/requirements.txt
WORKDIR .
COPY . .

RUN apk add nginx
COPY ./nginx.conf /etc/nginx/nginx.conf
COPY ./nginx.conf /etc/nginx/conf.d/default.conf
COPY ./nginx.conf /etc/nginx/sites-enabled/nginx.conf
COPY ./nginx.conf /etc/nginx/templates/default.conf.template

RUN mkdir static
RUN chmod 777 -R static
RUN chmod 777 -R /var/lib/nginx
RUN chmod 777 -R /var/log/nginx
RUN chmod 777 -R /run/nginx
RUN chmod 777 -R /etc/nginx


RUN pip install -r requirements.txt

ENV POSTGRES_USER=postgres
ENV POSTGRES_PASSWORD=postgres
ENV POSTGRES_DB=postgres
ENV POSTGRES_HOST=0.0.0.0
ENV POSTGRES_PORT=5432

RUN apk add postgresql-client
RUN mkdir -p /run/postgresql && chown -R postgres:postgres /run/postgresql
RUN mkdir -p /var/lib/postgresql/data && chown -R postgres:postgres /var/lib/postgresql/data

USER postgres
RUN initdb -D /var/lib/postgresql/data \
    && pg_ctl start -D /var/lib/postgresql/data \
    && pg_ctl stop -D /var/lib/postgresql/data

ENV DJANGO_SUPERUSER_USERNAME=admin
ENV DJANGO_SUPERUSER_EMAIL=admin@example.com
ENV DJANGO_SUPERUSER_PASSWORD=admin

CMD pg_ctl start -D /var/lib/postgresql/data \
    && python manage.py migrate \
    && python manage.py shell -c "from django.contrib.auth import get_user_model; User = get_user_model(); User.objects.filter(username='admin').exists() or User.objects.create_superuser('admin', 'admin@example.com', 'admin')" \
    && python manage.py collectstatic \
    && nginx \
    && gunicorn -b 0.0.0.0:8000 pr.wsgi:application

EXPOSE 80
EXPOSE 5432
EXPOSE 8000