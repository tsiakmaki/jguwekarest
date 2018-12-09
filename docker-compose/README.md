# JGU WEKA Rest Service and docker-compose 

A docker-compose example of setting the jgu weka rest servive with pre-existing certificate for securing the connection.  

1. Prepare your certificates (i.e. *.pem files) and update Dockerfile (if needed)  
2. Edit mongo db hostname with your hostname  
3. Run  
```
docker-compose build
docker-compose create
docker-compose start
```
to create the docker processes.  
4. Verify the service https://yourhostname:8443/weka_rc  
5. Make test post, e.g.  
```
curl -X POST -H "Content-Type: multipart/form-data" -H "Accept:text/uri-list" -F "file=@test.arff;" -F "estimatorParams=0.5"  -F "searchAlgorithm=local.K2" -F useADTree=0 -F "estimator=SimpleEstimator" -F searchParams='-P 1 -S BAYES' https://yourhostname:8443/weka_rc/algorithm/NaiveBayes
```
and verify that the task is complete using the returned task url. 
6. Watch logs at /usr/local/tomcat/logs/*   
```
docker exec -ti weka /bin/bash
vim /usr/local/tomcat/logs/catalinaxxx
``` 
