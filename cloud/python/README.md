
Cloud Function Run:
------------------

For any changes made to main.py or templates/getfractal.html, the cloud function will need to be deployed again.
Do not make any changes to the function name inside main.py


Deploy function getfracta1
--------------------------
gcloud functions deploy getfractal1 --runtime python39 --trigger-http --allow-unauthenticated
