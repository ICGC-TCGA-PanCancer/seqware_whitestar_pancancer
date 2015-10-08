## Users - running the container

1. Set permissions on datastore which will hold results of workflows after they run
        
        mkdir /workflows && mkdir /datastore
        chmod a+wrx /workflows && chmod a+wrx /datastore

2. Run the tabix server as a named container if you have not already (see the [tabix](https://github.com/ICGC-TCGA-PanCancer/pancancer_tabix_server)) 


3. Download and expand your workflows using the SeqWare unzip tool, it requires Java 7. Here we use Sanger as an example (you should probably pick a shared directory outside of this directory to avoid interfering with the Docker context if you need to rebuild the image). 

         cd /workflows
         wget https://seqwaremaven.oicr.on.ca/artifactory/seqware-release/com/github/seqware/seqware-distribution/1.1.1/seqware-distribution-1.1.1-full.jar
         wget https://s3.amazonaws.com/oicr.workflow.bundles/released-bundles/Workflow_Bundle_SangerPancancerCgpCnIndelSnvStr_1.0.6_SeqWare_1.1.0.zip
         java -cp seqware-distribution-1.1.1-full.jar net.sourceforge.seqware.pipeline.tools.UnZip --input-zip Workflow_Bundle_SangerPancancerCgpCnIndelSnvStr_1.0.6_SeqWare_1.1.0.zip --output-dir  Workflow_Bundle_SangerPancancerCgpCnIndelSnvStr_1.0.6_SeqWare_1.1.0

4. Run container and login with the following (while persisting workflow run directories to datastore, and opening a secure link to the tabix server). Here we assume that a tabix container has already started, that you want to store your workflow results at /datastore and that the workflow that you wish to run (Sanger) is present in the workflows directory. Change these locations as required for your environment. For example, you can omit the link parameter if you are running a tabix server on a different host.

         docker run --rm -h master -t -v /datastore:/datastore -v /workflows/Workflow_Bundle_SangerPancancerCgpCnIndelSnvStr_1.0.6_SeqWare_1.1.0:/workflow -v /tmp/custom_workflow.ini:/workflow.ini   -i pancancer/seqware_whitestar_pancancer:1.1.1  seqware bundle launch --dir /workflow --no-metadata --ini /workflow.ini 

5. Create an ini file (the contents of this will depend on your workflow). For testing purposes, you will require the following ini, note that the ip address for the tabix server will appear in your environment variables as PANCANCER\_TABIX\_SERVER\_PORT\_80\_TCP\_ADDR 

         # not "true" means the data will be downloaded using AliquotIDs
         testMode=true
         # the server that has various tabix-indexed files on it, see above, update with your URL
         tabixSrvUri=http://172.17.0.13/   

6. Run workflow sequentially (inside the container) with 

         seqware bundle launch --dir /workflow --no-metadata --ini workflow.ini

   Alternatively, run it in parallel with the following command. 
 
         seqware bundle launch --dir /workflow --no-metadata --ini workflow.ini --engine whitestar-parallel

7. For running real workflows, you will be provided with a gnos pem key that should be installed to the scripts directory of the Sanger workflow. Note that you can change the path of /workflows and /datastore on your host as needed in your environment.

8. Please note that you can re-use workflow.ini files by creating them outside the container and mounting them inside the container. 
    
    vim /tmp/custom_workflow.ini (modify the ini above for your environment)
    docker run --rm -h master -t --link pancancer_tabix_server:pancancer_tabix_server -v /datastore:/datastore -v /workflows/Workflow_Bundle_SangerPancancerCgpCnIndelSnvStr_1.0.6_SeqWare_1.1.0:/workflow -v /tmp/custom_workflow.ini:/workflow.ini   -i pancancer/seqware_whitestar_pancancer:1.1.1  seqware bundle launch --dir /workflow --no-metadata --ini /workflow.ini

## Developers - building the image locally  

1. Assuming docker is installed properly, build image with 
 
        docker build  -t pancancer/seqware_whitestar_pancancer .
