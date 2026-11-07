---
weight: 2
---
# Overview

The [Mastodon][1] system is an open-source social media platform. Its interface
is inspired by Twitter, but unlike Twitter, it operates as a federated network
of Mastodon instances. In the CyberLab, the Mastodon system is usually
standalone, taking the place of traditional proprietary platforms such as
Twitter in CyberLab cyber ranges.

For automated interactions with Mastodon, consult the [Mastodon API][2]
documentation.

## 1. Setup

To deploy this system, add `mastodon` to a lab configuration that is being deployed or your `deployment_config.yml` file.

## 2. Usage

### 2.1 Web access

On your development machine, you can forward the Mastodon web interface
to your machine. (Make sure to replace `MASTODON_IP` with the IP address of
the Mastodon web server, and `USER@HOST` with your local jumpbox details.)

```
ssh -i cyberlab-jumpbox-ssh-key.pem -L 8080:MASTODON_IP:8080 -Nf USER@HOST
```

Alternatively, you can also access this via the CyberLab VPN.

You should now be able to access Mastodon at <http://localhost:8080>!

### 2.2 Admin account

Credentials to the admin account are as follows:

  - **Email:** `cyberlabadmin@localhost`
  - **Password:** *output of the following command on master.mastodon.lab:*

    ```bash
    kubectl logs -n applications $(kubectl get pods --field-selector=status.phase==Succeeded -n applications -o jsonpath='{.items[*].metadata.name}') | grep "New password" | awk '{print $3}'
    ```

### 2.3 Enable signups (optional)

If you want users to register accounts without approval, then in the admin
account, navigate to
**Preferences > Administration > Server settings > Registration**:

<!--![Mastodon signups settings](images/mastodon-signups.png)-->

Set **Who can sign-up** to "Anyone can sign up" and click **Save changes**.

### 2.4 User accounts (optional)

In one window, you will need to be logged into the admin account. To register a
regular user account, you need another (private) window.

Let the user fill in their details. The email address must be @localhost:

<!--![Mastodon registration](images/mastodon-register.png)-->

You will then get a message to check your inbox. Localhost has no inbox, and
Mastodon does not accept internal CyberLab inboxes, so the Mastodon admin must
manually approve signups.

In the admin account, navigate to **Preferences > Moderation > Accounts**. You
should see the new user (greyed out).

<!--![Mastodon accounts page](images/mastodon-accounts.png)-->

Click the user's username, then on the user's page, press **Confirm**:

<!--![Mastodon signup approval](images/mastodon-approve.png)-->

In the user's private window, they should now be registered and logged in.

If you want to add user accounts programmatically and/or en masse, consult the
[Mastodon API][2] documentation.

Additionally, it may be beneficial to look into the `tootctl` tool, especially when considering making changes to the setup of mastodon.

## 3. Contributors

Rishi Adhikari ([@rishiad][4]) and Jordan Chin ([@Ting-Wei-Chin][5]) added the
Mastodon system to the CyberLab.


[1]: https://github.com/mastodon/mastodon
[2]: https://docs.joinmastodon.org/methods/
[3]: https://github.com/UAdelaide/CyberLab/tree/env/mastodon
[4]: https://github.com/rishiad
[5]: https://github.com/Ting-Wei-Chin
