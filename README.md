# JENKINS-PIPELINE
PIPELINE FOR DB DUMP DEPLOYMENT PROCESS 

Contains jenkins file for full DB dump deployment process which automates the  entire process of bringing env down , taking DB dump on source Oracle DB  server transeferring it to target Oracle DB server , bringing target environment down , taking target DB dump backup ,importing dump to target DB and finally restarting target environment . 
