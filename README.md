# Bind Private DNS Server Using Docker

The files within allow for running a private DNS server using Bind inside a Docker container. This is based on the following [project](https://hub.docker.com/r/sameersbn/bind/) from **sameersbn**.

Also, if you wish to setup a highly available complete CICD solution running in Azure using this solution, see [this article](https://danieleagle.com/2017/10/setting-up-a-private-cicd-solution-in-azure/). It contains a plethora of information that will greatly complement the text within. The Azure specific Bind files can be found in the [azure](./azure/) folder within this repository.

## Latest Changes

Be sure to see the [change log](./CHANGELOG.md) if interested in tracking changes leading to the current release.

## Getting Started

1. Create a Docker network named `main` by typing the following command (geared toward Linux):

    `sudo docker network create main`

2. In the same file, edit the **ROOT_PASSWORD** environment variable and change it to the desired password.

3. Run the following command (geared toward Linux) in the root of where the this repository was cloned:

    `sudo docker-compose up -d`

4. Open the Webmin interface and navigate to the Bind settings page. The username will be **root** and the password the value you assigned previously.

5. Under `Access Control Lists`, add a ACL named **trusted** or any preferred name with the following (depending on network configuration):

    ``` bash
    192.168.1.0/24
    127.0.0.1
    ```

6. Navigate to `Zone Defaults` and then under the `Default Zone Settings` section change `Allow Queries From` to `Listed`. In the edit box specify the ACL created earlier in the previous step (e.g. **trusted**). Now click `Return to zone list`.

7. Click on `Edit Config File` then on the **Edit config file** drop down, select `named.conf.options` and then click the **Edit** button. This is where some important options will be configured to ensure this DNS server truly is private.

    Ensure your **named.conf.options** configuration file looks like the code below.

    ```bash
    options {
      directory "/var/cache/bind";

        dnssec-enable yes;                        # enables DNS Security Extensions
        dnssec-validation auto;                   # indicates that a resolver (a caching or caching-only name server) will attempt to validate
                                                  # replies from DNSSEC enabled (signed) zones

        recursion yes;                            # allows recursive queries
        allow-recursion { trusted; };             # allows recursive queries only from clients defined in the "trusted" acl
        allow-query { trusted; };                 # allows queries only from clients defined in the "trusted" acl
        allow-transfer { none; };                 # do not allow zone transfers
        auth-nxdomain no;                         # conform to RFC1035
        version "Elvis has left the building.";   # hides the version information of Bind enabling security by obscurity

      forwarders {
        8.8.8.8;
        8.8.4.4;
        };
    };
    ```

    Feel free to change the forwarders to different DNS servers if desired (currently it's set to Google's Public DNS).

8. Click on the `Save` button and then `Apply Configuration`. Now click `Return to zone list`.

9. Test the DNS server is working by typing (geared toward Linux) `host google.com 192.168.1.50` (replace the IP address with the one used by your machine running Docker). If a response is received, you are ready to continue.

10. Create a Reverse Zone by clicking on `Create Master Zone`. For **Zone Type**, select **Reverse**.

11. In the **Domain name / Network** box, enter the IP address of your DNS server: `192.168.1.50` (replace the IP address with the one used by your machine running Docker). Enter `ns1.internal.example.com` (replace **example.com** with the domain of your choice) in the **Master server** box. Enter the email address of the DNS administrator in the **Email address** box below. Now click the **Create** button. Now click `Return to zone list`.

12. Create a Forward Zone by clicking on `Create Master Zone`. For **Zone Type**, select **Forward**. In the **Domain name / Network** box, enter your domain: `internal.example.com` (replace **example.com** with the domain of your choice). In the **Master server** box, enter `ns1.internal.example.com` (replace **example.com** with the domain of your choice). Enter the email address of the DNS administrator in the **Email address** box below. Now click the **Create** button.

13. Click `Address` to create a new DNS record. In the **Name** box, enter `ns` and then in the **Address** box, enter `192.168.1.50` (replace the IP address with the one used by your machine running Docker). Click the **Create** button.

14. Create applicable A records for the hosts of your choice (e.g. **webserver.internal.example.com**) by clicking on `Address` again. Enter the desired name (e.g. **nas**) and its IP address then click **Create**. The name will be added to the list as `host.internal.example.com` (e.g. **nas.internal.example.com**).

15. After entering in all applicable information, click on `Apply Configuration`. This should update the DNS server to use the new updated settings.

16. Finally, change the system time to the correct value in Webmin by going to the search bar and typing **System Time** and then clicking on the relevant search result.

## Docker Network

This image uses a network named `main`. Be sure to create that network by typing `sudo docker network create main` before running `docker-compose up -d`.

## Webmin Interface

This image uses Webmin to manage the Bind DNS server settings. It can be accessed by going to `https://192.168.1.50:10001` (replace the IP address with the one used by your machine running Docker). The default username is `root` and the default password is `password` (unless changed by the **ROOT_PASSWORD** environment variable in docker-compose.yml).

## Logs

Currently, logs can be found by going to `/var/lib/docker/containers/container-id/`. This is due to the way logs are being written (e.g. using -g option to run the server in the foreground and force all logging to stderr). See [this article](https://linux.die.net/man/8/named) for more details.

## Further Reading

For additional information, read [this article](http://www.damagehead.com/blog/2015/04/28/deploying-a-dns-server-using-docker/).
