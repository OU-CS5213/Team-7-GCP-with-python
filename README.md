Team 7
======

Members:
-------
Aditya K Kasturi

Aditya Rohan Singh

Naveen Kumar Gorantla

Pradipkumar Rajasekaran

Webpage Static URL
------------------
https://34.102.205.200

Cloud Function URL
------------------
https://us-central1-gold-episode-347200.cloudfunctions.net/getfractal1


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


Steps Executed to access the webpage via static IP:
===================================================

Documentation used:

Load Balancer with Cloud Function
-------------
https://cloud.google.com/load-balancing/docs/https/setting-up-https-serverless#gcloud_1

Self Signed SSL Certificate
---------------------------
https://cloud.google.com/load-balancing/docs/ssl-certificates/self-managed-certs

1. Reserve an external IP address

gcloud compute addresses create fractaldisplay --network-tier=PREMIUM --ip-version=IPV4 --global


2. Create a serverless NEG for your serverless app with cloud function

gcloud compute network-endpoint-groups create fractal --region=us-central1 --network-endpoint-type=serverless --cloud-function-name=getfractal1

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

gcloud compute backend-services create get-fractal --load-balancing-scheme=EXTERNAL --global

5. Add the serverless NEG as a backend to the backend service

gcloud compute backend-services add-backend get-fractal --global --network-endpoint-group=fractal --network-endpoint-group-region=us-central1

6. Create a URL map to route incoming requests to the backend service:

gcloud compute url-maps create fractal-map --default-service get-fractal

7. Create a target HTTP(S) proxy to route requests to your URL map

gcloud compute target-https-proxies create getfractalgcp --global --url-map fractal-map --global-url-map --ssl-certificates fractal-certificate --global-ssl-certificates

8. Create a forwarding rule to route incoming requests to the proxy

gcloud compute forwarding-rules create fractalforwarding --load-balancing-scheme=EXTERNAL --network-tier=PREMIUM --address=fractaldisplay --target-https-proxy=getfractalgcp --global --ports=443

9. Verify the load balancer is working by opening the below IP address:

https://34.102.205.200
 
