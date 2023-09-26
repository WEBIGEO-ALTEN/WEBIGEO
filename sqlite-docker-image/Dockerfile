FROM python:3.9
ARG BRANCH=dev
RUN apt update && apt install git 

RUN git clone --branch=$BRANCH https://github.com/WEBIGEO-ALTEN/WEBIGEO_BACK.git
#RUN cd /WEBIGEO_BACK/ && git pull

#RUN mkdir WEBIGEO && cd WEBIGEO_BACK/ && mv db.sqlite3 db.sqlite3.bak && cp -r * /WEBIGEO/
WORKDIR WEBIGEO_BACK/

RUN pip install --no-cache-dir -r requierements.txt
EXPOSE 8000
CMD ["python3", "manage.py","runserver", "0.0.0.0:8000"]


