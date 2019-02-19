* # Deployment

  ## Before you start

  Before you start, we recommend that you review the documentation of [proxy settings](proxy.md) and the documentation of [firewall settings](firewall.md). You can avoid problems related to the configuration of this modules by reading and applying the considerations mentioned there.

  ## Steps to deploy

  1. Have a machine with CentOS 7 and git

  2. Make sure that there is a password set for root and the user that will execute the scripts

  3. Clone the deployment repository

     `git clone https://github.com/JuezUN/Deployment.git`

  4. Inside the Deployment folder, make the .sh files runnable

     `chmod +x *.sh`

  5. Disable selinux

     `sudo ./disable_selinux.sh`

     *Running this command will cause the server to restart automatically so that the changes are applied*

  6. Install the prerequisites, you can use `./install_prerequisites.sh` to do this. 
     ​      
       *Make sure you DON'T run this command with sudo as the user is the one that should be able to use the applications (not sudo)*

  7. Logout and log back in to the server so that the user can use mongo and docker without `sudo`.

  8. Run the command `./setup_environment.sh && source env.sh` to set the environment variables (such as proxy and ports used by the backend microservices).

  9. Modify the `configuration.yaml` file to use the setup you want.

  If you want to deploy the entire application in a single machine, then

  - Execute the script `./run.sh`

  Otherwise, if you want to separate the grading machines from the rest of the application

  - Have *n>=1* machines with CentOS 7 that can communicate to the main machine via tcp protocol

  - In `configuration.yaml` in the backend option, set it to `backend: tcp://127.0.0.1:2000`

  - Run `sudo ./install_backend_service.sh` to setup the backend as a systemd service so that you don't have to manage it manually.

    *You will be requested a password for the backend user, this user is the one who is going to own the backend service*

    *Running this command will enable the backend service, which means that even after reboot it will be run by the init procedure.*

  - Run `./run.sh --distributed` to deploy the lighttpd, nginx and the backend service. You should be able to have the application working by now (except for the submission grading)

  - Go to [Grading host deployment documentation](https://github.com/JuezUN/Deployment/tree/master/grader-host) to deploy any number of hosts that will be used as grading machines.

  # Configuration

  ## Configuration.yml

  This file specifies which plugins will run when the judge deploys, also is needed that the settings for the SMTP server are set properly (i.e. username and password), if you want to.

  # What to do if server reboots

  When you reboot the server, you will see that the front end will work fine but if you try  to submit code it won't work. To fix that you just have to follow next few steps:

  1. Go into the *Deployment* directory.
  2. Run the command `./setup_environment.sh && source env.sh` to set the environment variables (such as proxy and ports used by the backend microservices).
  3. If you are deploying the agent in a single machine, run the command `sudo bash deployment_scripts/build_all_containers.sh`
  4. Run the command `uncode_full_restart`in order to restart all services and make everything work correctly.

  You are all set, try submitting code and see if it works.

  # Common problems

  There are some problems that you might find when deploying the services. 

  - Docker compose says that the ports of the micro services are not specified. Solution: Make sure you run the command `source env.sh`
  - Mongo DB fails to start after a reboot. There is an unsolved issue with systemd and mongod service that prevents it from starting correctly at boot. Until this issue is solved, the following workaround will start mongodb as INGInious needs.
    `sudo mongod -f /etc/mongod.conf`