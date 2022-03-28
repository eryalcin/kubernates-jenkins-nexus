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


# 1-Vagrant Kurulumu ve VirtualBox Entegresi

Vagrant(https://www.vagrantup.com/) toolunu sanallaştırma ortamımızda kubernates ortamını konuşlandıracağımız VM leri hazırlamak için kullanacağız.

Bu projede sanallaştırma altyapısı için Vindows10 üzerinde VirtualBox tercih edilmiştir. 
https://ugurakgul.medium.com/creating-a-local-kubernetes-cluster-with-vagrant-ba591ab70ee2 linkinden kurulumu yukarıdaki VirtualBox ve Vagrant versiyonları için değiştirip kendi ortamınıza kurulumu sağlayabilirsiniz.

![image](https://user-images.githubusercontent.com/32457347/160400160-71f9e97a-b4f2-4da4-ae72-9dfde9befe19.png)

# Ansible Kurulumu

Ben bu çalışmayı kaynak eksikliğinden dolayı VirtualBox üzerinden devam ettiremedim. OpenStack ortamında Ubuntu18.04  sunucularında gerçekleştireceğim.
Master node olacak VM'e ansible 2.10.15 kurulumu yapıyoruz.

![image](https://user-images.githubusercontent.com/32457347/160401804-827e5f2a-ae4e-4d43-b017-c6d8af94a0e2.png)


$ sudo apt update

$ sudo apt upgrade

$ sudo apt install software-properties-common

$ sudo apt-add-repository ppa:ansible/ansible

Ansible ile ilgili kurulumları devam ettirmek için ansible sunucusundan diğer sunucularımıza ssh ile ilgili bir ayar yapmamız gerekiyor.

https://topkaya.medium.com/sshla-%C5%9Fifre-girmeden-uzak-ba%C4%9Flant%C4%B1-kurmak-e8f4d28a01a3 Linkinden bu ayarlamayı takip edebilirsiniz.

# Kubespray ile K8s Ortamının Kurulumu

Bu bölüm için https://mydevops353097059.wordpress.com/kubespray-tr/ adresinden kopyalanmıştır.

![image](https://user-images.githubusercontent.com/32457347/160402048-9498ff5b-fd90-453b-ac78-f95f73dbfc3c.png)

Bu bölümde  kubespray kullanarak 1 master ve 2 node’u olan kubernetes clusterı oluşturacağız. Ben bu çalışmayı kaynak eksikliğinden dolayı VirtualBox üzerinden devam ettiremedim. OpenStack ortamında Ubuntu18.04  sunucularında gerçekleştireceğim.

# Kubespray Nedir?

Kubespray kendi sunucularımıza veya cloud sunucularına (aws gibi) ansible kullanarak kubernetes clusterı oluşturmamıza yarayan bir tooldur.

Cluster için Gerekli Uygulama Sunucuları

3 adet 18.04(4vCPU-8GB_Ram-80GB_Disk) sunucusu (1 master 2 node)

# Kubespray Kurulumu

master sunucusuna kubespray kurmak için aşağıdaki komutu çalıştırmanız yeterli olacaktır.

git clone https://github.com/kubernetes-incubator/kubespray.git

KubeSpray git reposunu sunucumuza indirdik. Daha sonra yapılması gereken klasörün içine girildikten sonra  requirements.txt dosyasındaki modülleri kurmamızdır. Bunları teker teker kurmak yerine pip kullanarak kurulumu gerçekleştireceğiz.

cd kubespray/

apt-get install python-pip

pip install -r requirements.txt

Komutları çalıştırdıktan sonra kubespray kurulumunu tamamladık. Bu kurulumdan ziyade daha çok kubesprayı hazırlamak gibi düşünebiliriz.

# Kubespray Konfigürasyonu

Kubesprayi hazırladıktan sonra geriye kubesprayi çalıştırmak kalıyor. Çalıştırmadan önce bir kaç tane düzenleme yapmamız gerekecek.

Öncelikle kubernetes clusterı kuracağımız master ve node sunucularımızda aşağıdaki komutları çalıştırınız.

sysctl -w net.ipv4.ip_forward=1

ufw disable

Yukarıdaki komutlardan sonra kubepsray sunucumuzda kubernetes cluster sunucularımıza ssh bağlantısı yaparken kullandığımız pem dosyasını kaydedelim.

vi k8s.pem

chmod 600 k8s.pem

Bu işlemden sonra kendi ansible inventory’imizi oluşturmamız gerekmektedir. Bunun için kubespray/inventory/sample klasörünü kopyalamamız gerekecek.

cp -rfp inventory/sample/ inventory/mycluster

Daha sonra inventory builder kullanarak host dosyamızı editlememiz gerekecek.

declare -a IPS=(10.10.10.11 10.10.10.12 10.10.10.13)

CONFIG_FILE=inventory/mycluster/hosts.ini python contrib/inventory_builder/inventory.py ${IPS[@]}

*Not1: Yukarıdaki ip’leri kendi ip’leriniz ile değiştirmeyi unutmayınız. 

Inventory’imizi update ettikten sonra hosts.ini klasörünü editlememiz gerekecektir.

cd /home/ubuntu/kubespray/inventory/mycluster

vi hosts.ini

Sizlerde hosts.ini dosyasını aşağıdaki gibi düzenleyebilirsiniz.

[k8s-cluster:children]

kube-master

kube-node

[all]

master ansible_host=10.10.10.11

node1 ansible_host=10.10.10.12

node2 ansible_host=10.10.10.13


[kube-master]

master

[kube-node]

node1

node2


[etcd]

master

node1

node2


[calico-rr]


[vault]

node1

node2

node3


[all:vars]

ansible_python_interpreter=/usr/bin/python3

ansible_user=ubuntu

ansible_become=yes


Yukaridaki işlemlerden sonra artık clusterımızı yaratabiliriz.

ansible-playbook -i inventory/mycluster/hosts.ini cluster.yml –private-key=k8s.pem

Yukarıdaki komut ile birlikte cluster oluşturma işlemi başlayacaktır. Bu işlem biraz uzun sürebilir.

İşlem bittikten sonra clusterımız hazır olacaktır.

# Kunernetes te Jenkins Ayaklandırma 
Kurulum için için https://github.com/eryalcin/kubernates-jenkins-nexus/blob/43a665524ce710612d1c361e4169be07b0239fb6/setup-Jenkins-on-Kubernetes dosyasına bakınız 

# Kunernetes te Nexus Ayaklandırma 
Kurulum için için nexus-deploy dosyasına bakınız 
