Team 7
======

Members:
-------
Aditya K Kasturi

Aditya Rohan Singh

Naveen Kumar Gorantla

Pradipkumar Rajasekaran

Webpage Static URL For Cloud function implementation:
----------------------------------------------------

https://34.102.205.200

Webpage Static URL for Cloud run implementation:
-----------------------------------------------------

https://34.110.171.39 (Not working currently)

Cloud Function URL
------------------
https://us-central1-gold-episode-347200.cloudfunctions.net/getfractal1


Cloud Run URL
------------
https://getfractal1-47ntsdicza-uc.a.run.app


Screenshots from Tashfeen's Website:
-----------------------------------

https://tashfeen.org/fractalsetc/build/index.html

Files used
==========
cloud/python/main.py

cloud/python/templates/getfractal.html

cloud/python/cert


Steps Executed to create cloud function:
=======================================

1. Cloned the repo provided for reference to local machine.

2. Removed unused files.

3. Selected cloud function as method of implementation.

4. Edited main.py and added function getfractal1 to create a cloud function called getfractal1 which triggers flask framework to run a html page.

5. Created a storage bucket and stored screenshots from Tashfeen's website. Made it accessible to public.

6. Enable Cloud function API.

7. Created a html page under templates called getfractal.html and added code to display all the images stored in the storage bucket..

8. Deployed the cloud function via command line from GCP instance.

gcloud functions deploy getfractal1 --runtime python39 --trigger-http --allow-unauthenticated

9. Added allUsers under permission via console with the role "Cloud Function Invoker" to the function getfractal1

10. Verified deployment of getfractal1 by opening the url found under trigger.

https://us-central1-gold-episode-347200.cloudfunctions.net/getfractal1

11. Cloud function deployment of html page complete.


Steps Executed to create cloud run:
==================================

1. Same steps from 1 to 5 from cloud function.

2. Enabled Cloud Run API and artifact registry API.

3. Same as step 7 from above.

4. Edit Dockerfile and replace FROM python:2-slim with FROM python:3-slim

5. Uncomment gunicorn from requirements.txt

6. Uncomment the ENTRYPOINT entry for gunicorn replacing the default entry in Dockerfile

7. Deployed the cloud run via command line from GCP instance

gcloud run deploy getfractal1 --source . --allow-unauthenticated --platform managed

8. Added allUsers under permission via console with the role "Cloud Run Invoker" to the run getfractal1/

9. Verified Deployment of cloud run gerfractal1 by opening the url found under trigger.

https://getfractal1-47ntsdicza-uc.a.run.app

10. Cloud Run deployement of html page complete.


Steps Executed to access the webpage via static IP:
===================================================

-> Same steps will be executed for both Cloud function and cloud run except where we link the function/run to the load balancer.

Documentation used:

Load Balancer with Cloud Function
-------------
https://cloud.google.com/load-balancing/docs/https/setting-up-https-serverless#gcloud_1

Self Signed SSL Certificate
---------------------------
https://cloud.google.com/load-balancing/docs/ssl-certificates/self-managed-certs


Steps
-----

1. Reserve an external IP address

->Cloud Function

gcloud compute addresses create fractaldisplay --network-tier=PREMIUM --ip-version=IPV4 --global

->Cloud run

gcloud compute addresses create fractaldisplay1 --network-tier=PREMIUM --ip-version=IPV4 --global

2. Create a serverless NEG for your serverless app with cloud function

->Cloud Function

gcloud compute network-endpoint-groups create fractal --region=us-central1 --network-endpoint-type=serverless --cloud-function-name=getfractal1

->Cloud run

gcloud compute network-endpoint-groups create fractal1 --region=us-central1 --network-endpoint-type=serverless --cloud-run-service=getfractal1

3. Create an SSL certificate resource

	3.1. Create a private key and certificate in PEM format. 
		Private Key file name: privatekey
		Certificate file name: certificate

	openssl genrsa -out /home/adirohan95/Software/cloudtest/cloud-nebulous-serverless/cloud/python/cert/privatekey 2048

	3.2. Create a certificate signing request (CSR) in the PEM format
		
		3.2.1. Create an OpenSSL configuration file called CONFIG_File

		3.2.2. Execute the blow command to generate the CSR

		openssl req -new -key privatekey -out CSR -config CONFIG_FILE

		3.2.3. Sign the Certificate
		openssl x509 -req -signkey privatekey -in CSR -out certificate -extfile CONFIG_FILE -extensions extension_requirements -days 100
	
	3.3. Create SSL certificate in cloud via command line

	gcloud compute ssl-certificates create fractal-certificate --certificate=certificate --private-key=privatekey --global

4. Create a backend service

->Cloud Function

gcloud compute backend-services create get-fractal --load-balancing-scheme=EXTERNAL --global

->Cloud Run
gcloud compute backend-services create get-fractal1 --load-balancing-scheme=EXTERNAL --global


5. Add the serverless NEG as a backend to the backend service

-> Cloud Function

gcloud compute backend-services add-backend get-fractal --global --network-endpoint-group=fractal --network-endpoint-group-region=us-central1

-> Cloud Run

gcloud compute backend-services add-backend get-fractal1 --global --network-endpoint-group=fractal1 --network-endpoint-group-region=us-central1

6. Create a URL map to route incoming requests to the backend service:

->Cloud Function

gcloud compute url-maps create fractal-map --default-service get-fractal

->Cloud Run 

gcloud compute url-maps create fractal-map1 --default-service get-fractal1


7. Create a target HTTP(S) proxy to route requests to your URL map

->Cloud Function 

gcloud compute target-https-proxies create getfractalgcp --global --url-map fractal-map --global-url-map --ssl-certificates fractal-certificate --global-ssl-certificates


->Cloud Run

gcloud compute target-https-proxies create getfractalgcp1 --global --url-map fractal-map1 --global-url-map --ssl-certificates fractal-certificate --global-ssl-certificates

8. Create a forwarding rule to route incoming requests to the proxy

->Cloud Function

gcloud compute forwarding-rules create fractalforwarding --load-balancing-scheme=EXTERNAL --network-tier=PREMIUM --address=fractaldisplay --target-https-proxy=getfractalgcp --global --ports=443

->Cloud Run

gcloud compute forwarding-rules create fractalforwarding1 --load-balancing-scheme=EXTERNAL --network-tier=PREMIUM --address=fractaldisplay1 --target-https-proxy=getfractalgcp1 --global --ports=443

9. Verify the load balancer is working by opening the below IP address:

-> Cloud function

https://34.102.205.200

-> Cloud Run (Having issues so its not currently working)

https://34.110.171.39

Thank you 
