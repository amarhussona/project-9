# Continous Integration Pipeline For Tooling Website

## Install Jenkins server

First create an EC2 instance of ubuntu server 20.04 and name it Jenkins:

![instance](./images/01_create_instance.png)

Install JDK due to jenkins being a java application:

`sudo apt update`
`sudo apt install default-jdk-headless -y`

![update](./images/02_apt_update.png)
![jdk](./images/03_install_jdk.png)

Install Jenkins:
```
wget -q -O - https://pkg.jenkins.io/debian-stable/jenkins.io.key | sudo apt-key add -
sudo sh -c 'echo deb https://pkg.jenkins.io/debian-stable binary/ > \
    /etc/apt/sources.list.d/jenkins.list'
sudo apt update
sudo apt-get install jenkins
```
![install jenkins](./images/04_install_jenkins.png)

Make sure Jenkins is up and running:

`sudo systemctl status jenkins`

![status jenkins](./images/05_verify_jenkins.png)

By default Jenkins server uses TCP port 8080 – open it by creating a new Inbound Rule in your EC2 Security Group:

![inbound rule](./images/06_inbound_rule.png)

Now perform Jenkins setup from brower:

![brower](./images/07_jenkins_ipaddress.png)

Retrieve password from jenkins server:

`sudo cat /var/lib/jenkins/secrets/initialAdminPassword`

![jenkins password](./images/08_jenkins_password.png)

Then choose suggested plugins when prompted:

![plugins](./images/09_suggested_plugins.png)

Create admin and finsih configuration:

![admin user](./images/10_create_admin_and_finish.png)

Installation is complete

Now configure Jenkins to retrieve source codes from github using webhooks.

Enable webhooks in github repo settings:

![add webhook](./images/11_add_webhook.png)

Go to Jenkins web console, click "New Item" and create a "Freestyle project":

![freestyle project](./images/12_create_freestyle_project.png)

To connect your GitHub repository, you will need to provide its URL, you can copy from the repository itself. In configuration of your Jenkins freestyle project choose Git repository, provide there the link to your Tooling GitHub repository and credentials (user/password) so Jenkins could access files in the repository:

![connect repo](./images/13_add_repo_cred.png)

Save the configuration and click the "Build Now" button to see if configured right in the console output:

![build 1](./images/14_build_output.png)

Click "Configure" in the job/project and add these two configurations
Configure triggering the job from GitHub webhook:

![configure triggering](./images/15_build_triggers.png)

Configure "Post-build Actions" to archive all the files – files resulted from a build are called "artifacts":

![post-build actions](./images/16_post_build.png)

Now make some change in the README.MD file in the GitHub repository and push the changes to the master branch to test it works:

![change readme](./images/17_edit_readme.png)

You can see its results – artifacts, saved on Jenkins server:

![result build 2](./images/18_confirm_setup_works.png)

![confirm](./images/19_confirm_artifacts.png)

By default, the artifacts are stored on Jenkins server locally:

`ls /var/lib/jenkins/jobs/tooling_github/builds/<build_number>/archive/`

![archive](./images/20_verify_archive_in_server.png)

## Configure Jenkins to copy files to NFS server via SSH

Now we have our artifacts saved locally on Jenkins server, the next step is to copy them to our NFS server to /mnt/apps directory.

Install "Publish Over SSH" plugin. On main dashboard select "Manage Jenkins" and choose "Manage Plugins" menu item. On "Available" tab search for "Publish Over SSH" plugin and install it:

![install ssh plugin](./images/21_publish_over_ssh_plugin.png)
![plugin install](./images/22_plugin_installed.png)

Next configure the job/project to copy artifacts over to NFS server. Scroll down to Publish over SSH plugin configuration section and configure it to be able to connect to your NFS server. Test the configuration and make sure the connection returns Success:

![config plugin](./images/23_configure_plugin.png)

Save the configuration, open your Jenkins job/project configuration page and add another one "Post-build Action". Configure it to send all files produced by the build into our previouslys define remote directory:

![post-build action](./images/24_post_build_action.png)

Save this configuration and go ahead, change something in README.MD file in your GitHub Tooling repository:

![edit readme](./images/25_delete_changes_in_readme.png)

Webhook will trigger a new job and in the "Console Output" this was the resulting error:

![error](./images/26_error_in_build.png)

As pointed out in the console output, it looks like the issue is with permissions so perform the code shown in this image:

![permissions](./images/27_chmod_chown.png)

Now make changes in the README.MD file again and see if the build is successful with the files being transferred:

![build 4](./images/28_build4_success.png)

To make sure that the files in /mnt/apps have been udated – connect via SSH/Putty to your NFS server and check README.MD file:

`cat /mnt/apps/README.md`

![verify readme](./images/29_verify_readme_in_nfs.png)