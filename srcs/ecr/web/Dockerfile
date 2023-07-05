# syntax=docker/dockerfile:1

FROM python:3
ENV PYTHONDONTWRITEBYTECODE=1
ENV PYTHONUNBUFFERED=1
WORKDIR /apps
COPY apps /apps/

EXPOSE 8000/tcp

RUN chmod +x /apps/entrypoint.sh
CMD /apps/entrypoint.sh
