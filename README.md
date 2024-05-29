# AWX_INSTALL
AWX install on rocky9.4 min. using AWX image 1.1.4


This is the link to the webpage: https://www.kilsec.com/index.php/2023/02/13/install-awx-on-rocky-9-1/

**These are the directions. I will work on an ansible playbook to automate intstall.
**
1.
curl -sfL https://get.k3s.io | sh -

2.
sudo chmod 644 /etc/rancher/k3s/k3s.yaml

3.
sudo dnf install git -y

4.
curl -s "https://raw.githubusercontent.com/kubernetes-sigs/kustomize/master/hack/install_kustomize.sh"  | bash

5.
sudo mv kustomize /usr/local/bin/

6.
#vi kustomization.yaml  **************
echo "

apiVersion: kustomize.config.k8s.io/v1beta1

kind: Kustomization

resources:

  # Find the latest tag here: https://github.com/ansible/awx-operator/releases

  - github.com/ansible/awx-operator/config/default?ref=1.1.4 # Change to whatever version you want to deploy - 1.1.4 latest at time of writting
 
# Set the image tags to match the git version from above

images:

  - name: quay.io/ansible/awx-operator

    newTag: 1.1.4 # Change to whatever version you want to deploy - 1.1.4 latest at time of writting

# Specify a custom namespace in which to install AWX

namespace: awx


" > kustomization.yaml


7.
kustomize build . | kubectl apply -f -    

8.
chown student:student kustomization.yaml

9.
mv kustomization.yaml /home/student/

10.
kubectl get pod -n awx

11.
#vi awx.yaml  ***********************

echo " 

---

apiVersion: awx.ansible.com/v1beta1

kind: AWX

metadata:

  name: awx # This can be whatever you want to name it, if you are deploying multiple instances each should be unique

spec:

  service_type: nodeport

" > awx.yaml




echo ############################################################ edit kustomize.yaml again

12.
echo "

apiVersion: kustomize.config.k8s.io/v1beta1

kind: Kustomization

resources:

  # Find the latest tag here: https://github.com/ansible/awx-operator/releases

  - github.com/ansible/awx-operator/config/default?ref=1.1.4 # Change to whatever version you want to deploy - 1.1.4 latest at time of writting

  - awx.yaml

# Set the image tags to match the git version from above

images:

  - name: quay.io/ansible/awx-operator

    newTag: 1.1.4 # Change to whatever version you want to deploy - 1.1.4 latest at time of writting

# Specify a custom namespace in which to install AWX

namespace: awx


" > kustomization.yaml


echo ######################################################################## compile contaier and start build

13.
kustomize build . | kubectl apply -f -

14.
echo " put this in another tab"
*******************

kubectl logs -f deployments/awx-operator-controller-manager -c awx-manager -n awx

*******************

15.
echo "this is your inital login password."
echo " go to https://localhost or Machine IP:30080 "

kubectl get secret awx-admin-password -o jsonpath="{.data.password}" -n awx | base64 --decode ; echo

