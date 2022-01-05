# kafka-ocp
PROCEDIMENTOS PARA INSTALAÇÃO DO KAFKA
Para efeito de testes, recomendo a utilização da versão developer do OpenShift Cloud oferecido pela RedHat:

https://www.redhat.com/en/technologies/cloud-computing/openshift/try-it 

<H1>Passo a Passo</H1>
Acessar o menu Operators => Operators Hub => Filtrar por Red Hat Integration - AMQ Streams: 
![alt text](https://github.com/leobower/kafka-ocp/blob/main/Images/img1.JPG)

Ao clicar no Red Hat Integration - AMQ Streams vai abrir uma tela com um botão "Install", clique nesse botão e vai aparecer a tela abaixo, 
marque a opção "A specific namespace on the cluster" e selecione o projeto [kafka (prd), kafka-hml (hml) ]onde quer instalar o operator e clique em "Install": 

![alt text](https://github.com/leobower/kafka-ocp/blob/main/Images/img2.JPG)

Após clicar em "install" vai aparecer uma tela como abaixo, aguarde até que o botão "View Operator" apareça, quando ele aparecer significa que o operator está instalado e pronto para uso, clique nesse botão: 

![alt text](https://github.com/leobower/kafka-ocp/blob/main/Images/img3.JPG)

Na tela que vai surgir clique no "Create instance" para o recurso "Kakfa", se o link não estiver disponível, faça um reload da página (CTRL + R): 

![alt text](https://github.com/leobower/kafka-ocp/blob/main/Images/img4.JPG)

Na tela de criação do Kafka, clique na opção "YAML view"  e insira o conteúdo do arquivo abaixo (kafka.yaml) e clique em "Create":  
https://github.com/leobower/kafka-ocp/blob/main/kafka.yaml

![alt text](https://github.com/leobower/kafka-ocp/blob/main/Images/img5.JPG)

Vai aparecer a tela como abaixo, após clicar no "Create":

![alt text](https://github.com/leobower/kafka-ocp/blob/main/Images/img6.JPG)

Clique no menu Workloads -> Pods, caso o projeto não esteja selecionado, selecione o projeto "kafka-hml", e aguarde os pods do zookeeper e do kafka ficarem no status "Running": 

![alt text](https://github.com/leobower/kafka-ocp/blob/main/Images/img7.JPG)

<H1>Instalação do Kafka Connect via S2i</H1>

Acessar Operators => Installed Operators. Localizar o Red Hat Integration - AMQ Streams 

![alt text](https://github.com/leobower/kafka-ocp/blob/main/Images/img8.JPG)

No menu superior, localizar Kafka Connect Source to Image e clicar em CreateKafkaConnectrS2I 

![alt text](https://github.com/leobower/kafka-ocp/blob/main/Images/img9.JPG)

Na tela de criação do KafkaConnectS2I, clique na opção "YAML view"  e insira o conteúdo do arquivo abaixo (kafkaconnects2i.yaml) e clique em "Create":  

https://github.com/leobower/kafka-ocp/blob/main/kafkaconnects2i.yaml

Atentar para a cfg namespace – [kafka-hml para HML e kafka para para PRD]

Clique no menu Workloads -> Pods, caso o projeto não esteja selecionado, selecione o projeto "kafka", e aguarde o pod do kafkaconnectS2i ficar no status "Running": 

![alt text](https://github.com/leobower/kafka-ocp/blob/main/Images/img10.JPG)

<H1>Instalação do Debezium no Kafka Connect</H1>

Utilizar o CLI “OC” do Open Shift, realizando o login no Open Shift: 

PréReqs: 

Ter o OC instalado na máquina local.
Ter um usuário com acesso ADM no Open Shift 

Recuperar o comando de Login: 

![alt text](https://github.com/leobower/kafka-ocp/blob/main/Images/img11.JPG)

Colando o comando de login na linha de comando: 

![alt text](https://github.com/leobower/kafka-ocp/blob/main/Images/img12.JPG)

Selecione o projeto Kafka: 

![alt text](https://github.com/leobower/kafka-ocp/blob/main/Images/img13.JPG)

Baixar os arquivos do Conector Debezium no site da Debezium, de acordo com o BD que se deseja capturar eventos de CDC PostgreSQL, SQL Server etc). Fazendo o download, descompactar e colocar em uma pasta física da máquina local: 
Site do Debezium: https://debezium.io/documentation/reference/stable/install.html 
Exemplo de arquivos para SQL Server - https://github.com/leobower/kafka-ocp/tree/main/Debezium/my_plugins/debezium-connector-sqlserver

Posicionar o CLI para a pasta onde estão os arquivos.

Resgatar o nome do Build Configuration do KafkaConnectS2i utilizando o comando abaixo (necessário estar logado no projeto do Kafka): 
[oc get bc] 

![alt text](https://github.com/leobower/kafka-ocp/blob/main/Images/img15.JPG)

Verifique o nome [my-connect-s2i-cluster-connect] e o utilize para rodar o seguinte comando: 

[oc start-build bc/my-connect-s2i-cluster-connect --from-dir ./my_plugins/ -n kafka] 

Onde 

my-connect-s2i-cluster-connect: nome do build configuration 

Kafka: namespace onde o Kafka foi instalado 

Esse comando utilizará os arquivos do Debezium para gerar um novo build do KafkaConnectS2I no OpenShift, copiando os arquivos do Conector Debezium da máquina local para o POD do OpenShift, possibilitando assim a instalação do Conector Debezium para o BD em questão, no exemplo aqui, é o SQL Server: 

![alt text](https://github.com/leobower/kafka-ocp/blob/main/Images/img16.JPG)

Verificar se no OpenShift foi disponibilizado uma nova versão do POD do KafkaConnectS2I 

![alt text](https://github.com/leobower/kafka-ocp/blob/main/Images/img17.JPG)

<H1>Instalação de Conectores Debezium no KafkaConnectS2I</H1>

Exemplo de yaml - https://github.com/leobower/kafka-ocp/blob/main/Debezium/CONECTORES_PRD/cenector-teste_debezium.yaml

![alt text](https://github.com/leobower/kafka-ocp/blob/main/Images/img17.JPG)

![alt text](https://github.com/leobower/kafka-ocp/blob/main/Images/img18.JPG)

![alt text](https://github.com/leobower/kafka-ocp/blob/main/Images/img19.JPG)

<H1>Instalação do Kafdrop</H1>

Acessar o menu Workloads => DeploymentConfigs, selecionar o projeto do Kafka e clicar em CreateDeploymentConfig: 

![alt text](https://github.com/leobower/kafka-ocp/blob/main/Images/img20.JPG)

Copiar o conteúdo arquivo na área de edição de Yaml, atentando para os valores destacados abaixo que vão variar de acordo com o ambiente e clicar em Create

![deploymentConfig](https://github.com/leobower/kafka-ocp/blob/main/kafdrop-deploymentconfig.yaml)

![alt text](https://github.com/leobower/kafka-ocp/blob/main/Images/img21.JPG)

![alt text](https://github.com/leobower/kafka-ocp/blob/main/Images/img22.JPG)

Acessar Workloads => Pods e verificar se o POD do kafdrop está em estado Running: 

<H1>Criando a Rota do Kafdrop </H1>
<H2>Service</H2>

Acessar Networking => Services, selecionar o projeto do Kafka e clicar em Create Service  

![alt text](https://github.com/leobower/kafka-ocp/blob/main/Images/img23.JPG)

Inserir o valor abaixo e clicar em Create: 

...

apiVersion: v1 
kind: Service 
metadata: 
  name: kafdrop 
  namespace: kafka 
spec: 
  selector: 
    io.kompose.service: kafdrop 
  ports: 
    - protocol: TCP 
      port: 80 
      targetPort: 9000 
      
...

![alt text](https://github.com/leobower/kafka-ocp/blob/main/Images/img24.JPG)

Verificar o mapeamento do POD do Kafdrop: 

![alt text](https://github.com/leobower/kafka-ocp/blob/main/Images/img25.JPG)

<H2>ROTA</H2>

Acessar o menu Network => Routes , selecionar o projeto do Kafka e clicar em CreateRoute 

![alt text](https://github.com/leobower/kafka-ocp/blob/main/Images/img26.JPG)


Na página de configuração setar as propriedades conforme abaixo: 

IMPORTANTE: PARA USO CORPORATIVO, O KAFDROP DEVE ESTAR FECHADO PARA ACESSO À INTERNET PARA ISSO É NECESSÁRIO UTILIZAR O WILDCARD PARA REDES INTERNAS

![alt text](https://github.com/leobower/kafka-ocp/blob/main/Images/img27.JPG)









