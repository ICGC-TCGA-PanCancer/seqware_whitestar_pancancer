## Users - running the container

1. Set permissions on datastore which will hold results of workflows after they run
        
        mkdir workflows && mkdir datastore
        chmod a+wrx workflows && chmod a+wrx datastore

2. Run the tabix server as a named container if you have not already (see the [tabix](../tabix)) 


3. Download and expand your workflows using the SeqWare unzip tool, it requires Java 7. Here we use Sanger as an example (you should probably pick a shared directory outside of this directory to avoid interfering with the Docker context if you need to rebuild the image). 

         cd workflows
         wget https://seqwaremaven.oicr.on.ca/artifactory/seqware-release/com/github/seqware/seqware-distribution/1.1.0-alpha.6/seqware-distribution-1.1.0-alpha.6-full.jar
         wget https://s3.amazonaws.com/oicr.workflow.bundles/released-bundles/Workflow_Bundle_SangerPancancerCgpCnIndelSnvStr_1.0.5.1_SeqWare_1.1.0-alpha.5.zip
         java -cp seqware-distribution-1.1.0-alpha.6-full.jar net.sourceforge.seqware.pipeline.tools.UnZip --input-zip Workflow_Bundle_SangerPancancerCgpCnIndelSnvStr_1.0.5.1_SeqWare_1.1.0-alpha.5.zip --output-dir  Workflow_Bundle_SangerPancancerCgpCnIndelSnvStr_1.0.5.1_SeqWare_1.1.0-alpha.5

4. Run container and login with the following (while persisting workflow run directories to datastore, and opening a secure link to the tabix server). Here we assume that a tabix container has already started, that you want to store your workflow results at /datastore and that the workflow that you wish to run (Sanger) is present in the workflows directory. Change these locations as required for your environment.  

         docker run --rm -h master -t --link pancancer_tabix_server:pancancer_tabix_server -v `pwd`/datastore:/datastore -v `pwd`/workflows/Workflow_Bundle_SangerPancancerCgpCnIndelSnvStr_1.0.5.1_SeqWare_1.1.0-alpha.5:/workflow  -i seqware/seqware_whitestar_pancancer 

5. Create an ini file (the contents of this will depend on your workflow). For testing purposes, you will require the following ini, note that the ip address for the tabix server will appear in your environment variables as PANCANCER\_TABIX\_SERVER\_PORT\_80\_TCP\_ADDR 

         # not "true" means the data will be downloaded using AliquotIDs
         testMode=true
         # the server that has various tabix-indexed files on it, see above, update with your URL
         tabixSrvUri=http://172.17.0.13/   

6. Run workflow sequentially with 

         seqware bundle launch --dir /workflow --no-metadata --ini workflow.ini

   Alternatively, run it in parallel with the following command. 
 
         seqware bundle launch --dir /workflow --no-metadata --ini workflow.ini --engine whitestar-parallel

7. For running real workflows, you will be provided with a gnos pem key that should be installed to the scripts directory of the Sanger workflow.

## Developers - building the image locally  

1. Assuming docker is installed properly, build image with 
 
    docker build  -t seqware/seqware_whitestar_pancancer .
