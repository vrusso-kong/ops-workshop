+++
title = ""
menuTitle = "Deploy OpsMan"
chapter = false
weight = 4
description = ""
draft = false
+++
## Goal
Utilizing Concourse and PCF Pipelines we are going to configure and deploy OpsMan to a new PCF Foundation.

### PCF Pipelines
Download the latest PCF-Pipeline from [PivNet](https://network.pivotal.io/products/pcf-automation/). If you can't access this...raise your hand and wait quietly.

Extract the `.tgz` file to our bbl-workshop directory and set our Azure storage container.

    $ export AZURE_STORAGE_ACCOUNT=$(bbl outputs | grep storage_account_name | cut -d " " -f2)
    $ export AZURE_STORAGE_ACCOUNT_KEY=$(az storage account keys list --account-name $AZURE_STORAGE_ACCOUNT| jq -r .[0].value)
    $ az storage container create --name terraformstate --account-name $AZURE_STORAGE_ACCOUNT

Pause, move into the PCF Azure version of the pipeline.

    $ cd pcf-pipelines/install-pcf/azure
    
 Now get ready for a whole bunch of Credhub copy/pasta.
 
    $ env -u CREDHUB_PROXY -u CREDHUB_SERVER -u CREDHUB_CLIENT -u CREDHUB_SECRET -u CREDHUB_CA_CERT bash
    $ credhub login -s https://$EXTERNAL_HOST:8844 -u admin -p $UAA_USERS_ADMIN_PASSWORD --skip-tls-validation 
    $ credhub set -n /concourse/main/azure_client_id -t value -v $BBL_AZURE_CLIENT_ID
    $ credhub set -n /concourse/main/azure_client_secret -t value -v $BBL_AZURE_CLIENT_SECRET
    $ credhub set -n /concourse/main/pivnet_token -t value -v <get your pivnet token from network.pivotal.io>
    $ credhub set -n /concourse/main/azure_storage_account_key -t value -v $AZURE_STORAGE_ACCOUNT_KEY
    $ credhub generate -n /concourse/main/opsman_admin_password -t password
    $ credhub generate -n /concourse/main/opsman_ssh -t ssh -m ubuntu
Trivia: We debated letting you guys figure this part out on your own. I vetoed that. You all owe me.

### Pretty Fly
    $ fly -t bbl-workshop set-pipeline -p install-pcf-azure -c pipeline.yml -l params.yml \
     --var "azure_client_id=((azure_client_id))" \
     --var "azure_client_secret=((azure_client_secret))" \
     --var "azure_region=$BBL_AZURE_REGION" \
     --var "azure_subscription_id=$BBL_AZURE_SUBSCRIPTION_ID" \
     --var "azure_storage_account_name=pcf" \
     --var "azure_buildpacks_container=buildpacks" \
     --var "azure_droplets_container=droplets" \
     --var "azure_packages_container=packages" \
     --var "azure_resources_container=resources" \
     --var "azure_tenant_id=$BBL_AZURE_TENANT_ID" \
     --var "azure_terraform_prefix=$(echo $AZURE_STORAGE_ACCOUNT | cut -c 7-20)" \
     --var "azure_vm_admin=ubuntu" \
     --var "company_name=MyCo" \
     --var "pcf_ert_domain=pcf.$(echo $AZURE_STORAGE_ACCOUNT | cut -c 7-20).io" \
     --var "system_domain=sys.pcf.$(echo $AZURE_STORAGE_ACCOUNT | cut -c 7-20).io" \
     --var "apps_domain=apps.pcf.$(echo $AZURE_STORAGE_ACCOUNT | cut -c 7-20).io" \
     --var "ert_major_minor_version=^2\.3\.[0-9]+$" \
     --var "opsman_admin_username=admin" \
     --var "opsman_admin_password=((opsman_admin_password))" \
     --var "opsman_domain_or_ip_address=opsman.pcf.$(echo $AZURE_STORAGE_ACCOUNT | cut -c 7-20).io" \
     --var "opsman_major_minor_version=^2\.3\.[0-9]+$" \
     --var "pcf_ssh_key_pub=((opsman_ssh.public_key))" \
     --var "pcf_ssh_key_priv=((opsman_ssh.private_key))" \
     --var "pivnet_token=((pivnet_token))" \
     --var "security_acknowledgement=X" \
     --var "azure_storage_container_name=terraformstate" \
     --var "terraform_azure_storage_access_key=((azure_storage_account_key))" \
     --var "terraform_azure_storage_account_name=$AZURE_STORAGE_ACCOUNT"

That was fun. Now that our concourse is properly configured we can use it to terraform and install opsman

    $ fly -t bbl-workshop unpause-pipeline -p install-pcf-azure
    $ fly -t bbl-workshop trigger-job -j install-pcf-azure/bootstrap-terraform-state -w
   (Will seem to hang on unpack-tarball step for 15 min or so)
    
    $ fly -t bbl-workshop trigger-job -j install-pcf-azure/create-infrastructure -w
    
At this point, with out proper DNS and Domain's defined, we cannot continue further. If anyone would like to donate a domain raise your hand and sit quietly.
