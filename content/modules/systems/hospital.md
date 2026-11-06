# Overview

The Hospital system is implemented by [OpenEMR][1] — a medical practice
management system that is open-source under the GPL (3.0). OpenEMR is certified
and used internationally in real clinics, making it ideal for simulating
hospitals in the CyberLab.

## 1. Setup

To deploy this system, include `hospital` in a lab configuration or the `deployment_config.yml`, then deploy the branch.

*(As of 5 June 2026, it takes about 19 mins to deploy a working system)*

## 2. Usage

You should be able to access OpenEMR at `http://localhost:8080`

Credentials to OpenEMR are randomly generated.

  - **Username:** `admin`
  - **Password:** *output of the following command:*

    ```bash
    echo $(kubectl get secret openemr-credentials -n applications -o jsonpath='{.data.admin-pass}' | base64 --decode)
    ```

## 3. Contributors

Nugroho Isworo ([@isworo00][5]) added the Hospital system to the CyberLab.


[1]: https://www.open-emr.org/
[2]: https://ronin.uoa.cloud
[3]: https://github.com/UAdelaide/Cyberlab/wiki/Kubernetes
[4]: https://fluxcd.io/
[5]: https://github.com/isworo00
