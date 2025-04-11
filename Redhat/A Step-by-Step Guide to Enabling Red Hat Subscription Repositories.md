## A Step-by-Step Guide to Enabling Red Hat Subscription Repositories  

1. Register Redhat Subscription

```bash
sudo subscription-manager register
```

It will prompt for username and password and enter the Redhat subscription manage website portal username and password

2. list available subscription

```bash
sudo subscription-manager list --available
```

3. attache the subscription

```bash
sudo subscription-manager attache --pool=<pool_id>
```

4. list available repos

```bash
sudo subscription-manager repos --list
```

5. enable repository

```bash
sudo subscription-manager repos --enable=<reponame or repo ID>
```

6. list enabled repos

```bash
sudo subscription-manager repos --list-enabled
```

7. update the repo

```bash
sudo dnf update
```

Additional:

8. Verify the subscription and repository status

```bash
sudo subscription-manager status
```

9 unregister and remove the subscription

```bash
sudo subscription-manager unregister
sudo subscription-manager remove --all
```