# Monitoreo-Sintetico-POP

Despliegue de Punto de Presencia Instana en clúster AWS EKS 
Se recomiendan las siguientes configuraciones para mejorar la seguridad en el entorno de producción.

Cifrado TLS

Para configurar TLS, necesita tener su par certificado-clave X.509 (tls.crt, tls.key) y el archivo del certificado raíz de la Autoridad de Certificación (CA) (ca.crt).
Si no dispone de sus archivos de claves y certificados, también puede utilizar el comando openssl para generar unos nuevos. 

1. Generar el certificado mediante openssl.

   Comando a ejecutar:
   
       openssl genrsa -out tls.key 4096
       openssl req -x509 -new -nodes -sha256 -key tls.key -days 3650 -subj '/O=Instana/CN=Certificate Authority' -out ca.crt
       openssl req -new -sha256 -key tls.key -subj '/O=Instana/CN=Server' | openssl x509 -req -sha256 -CA ca.crt -CAkey tls.key -CAserial ca.txt -CAcreateserial -days 3650 -out tls.crt

   Ejemplo:

       openssl genrsa -out tls.key 4096
       openssl req -x509 -new -nodes -sha256 -key tls.key -days 3650 -subj '/O=Redis/CN=Certificate Authority' -out ca.crt
       openssl req -new -sha256 -key tls.key -subj '/O=Redis/CN=Server' | openssl x509 -req -sha256 -CA ca.crt -CAkey tls.key -CAserial ca.txt -CAcreateserial -days 3650 -out tls.crt


2. Crear namespace para el despliegue del POP sintetico.

   Comando a ejecutar:
   
       kubectl create namespace syn

3. Crear el secreto donde se guardaran los certificados generados en los pasos previos.

   Comando a ejecutar:

       kubectl create secret generic pop-tls-secret -n syn --type='kubernetes.io/tls' --from-file=ca.crt=path/to/ca.crt --from-file=tls.crt=path/to/tls.crt --from-file=tls.key=path/to/tls.key

   Ejemplo:

       kubectl create secret generic syntheticpop-redis -n syn --type="kubernetes.io/tls" --from-file=ca.crt="/home/cloudshell-user/ca.crt" --from-file=tls.crt="/home/cloudshell-user/tls.crt" --from-file=tls.key="/home/cloudshell-user/tls.key"         


4. Desplegar el POP sintetico.


   Comando a ejecutar:

       helm install synthetic-pop --repo https://agents.instana.io/helm --namespace syn --create-namespace --set downloadKey="yourdownloadkey" --set controller.location="MyPoP;My PoP;MyCounty;MyCity;0;0;This is a testing Synthetic Point of Presence" --set controller.instanaKey="instanaAgentkey" --set controller.instanaSyntheticEndpoint="https://synthetics-green-saas.instana.io" --set redis.tls.enabled=true --set redis.tls.secretName="pop-tls-secret" synthetic-pop

   Ejemplo:

       helm install synthetic-pop --repo "https://agents.instana.io/helm" --namespace syn --create-namespace --set downloadKey="yourdownloadkey" --set controller.location="private_location;private_location;Peru;Lima;0;0;private_location" --set controller.clusterName="demo-eks" --set controller.instanaKey="yourdownloadkey" --set controller.instanaSyntheticEndpoint="https://synthetics-coral-saas.instana.io" --set redis.tls.enabled=true --set redis.tls.secretName="syntheticpop-redis" synthetic-pop


5. Verificar el POP en la consola de Instana

  ![image](https://github.com/juan-conde-21/Monitoreo-Sintetico-POP/assets/13276404/c3bb8d4a-57a4-44a1-88c0-25c247793dd2)








