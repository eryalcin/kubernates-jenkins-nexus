# kubernates-jenkins-nexus
Kubernetes'te Jenkins için Otomatik Dağıtım
#
Bu projemizde Kubernates ortmamında Jenkins ile otomatik dağıtım için ilgili Vagrant, Ansible, K8s ortamının kurulumu sağlanacak. Sonrasında k8s üzerinde Jenkins ve Nexus pod larımızı kuruyor olacağız.
#
Bu projede sanallaştırma altyapısı için Vindows10 üzerinde VirtualBox tercih edilmiştir. 
#

Bu proje 

VirtualBox 5.2.44

Vagrant 2.0.3

Ubuntu18.04 LTS

Ansible 2.10.15

Kubernetes v1.23.5 versiyonları ile test edilmiştir.


#
1-Vagrant Kurulumu

Vagrantı sanallaştırma ortamımızda kubernates ortamını konuşlandıracağımız VM leri hazırlamak için kullanacağız.

Bu projede sanallaştırma altyapısı için Vindows10 üzerinde VirtualBox tercih edilmiştir. 
https://ugurakgul.medium.com/creating-a-local-kubernetes-cluster-with-vagrant-ba591ab70ee2 linkinden kurulumu yukarıdaki VirtualBox ve Vagrant versiyonları için değiştirip kurulumu sağlayabilirsiniz.

