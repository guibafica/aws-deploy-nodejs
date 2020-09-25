Essa é uma aplicação bem simples que tem como principal funcionalidade testar e instruir o deploy de um backend na AWS. No final do processo, devemos poder acessar nossa aplicação que exibirá um "Hello Word" no navegador sem que precisemos  rodar a mesma em nossa máquina, então vamos lá!

> Referente a configuração deste aplicativo, não tem nada demais, apenas instalei o express e criei um script pra rodar o projeto. O mais básico possível para rodar um "Hello World!" na rota raiz...

Depois de configurar vamos ao deploy:

## 1º Passo - Dockerfile:
Precisamos criar esse arquivo na raiz `Dockerfile`, ele basicamente diz como nossa aplicação vai se comportar quando a gente for iniciar um novo servidor com essa aplicação.
 - FROM -> Qual versão do node a maquina virtual vai rodar;
 - WORKDIR -> O diretório onde estará a aplicação;
 - COPY -> Copiar os arquivos e por na pasta app;
 - RUN -> Qual comando vai executar;
 - EXPOSE -> Expor a porta que vai rodar a aplicação, a mesma que está no `index.js`;
 - CMD -> O comando que irá executar, "separado por virgula", quando a aplicação inicializar.

> Cria tbm o `.dockerignore` pra não copiar o node_modules. 

## 2º Passo - Rodar a aplicação com o docker-compose:
Agora, vamos criar mais um arquivo, o `docker-compose.yml`.
Nele, vamos configurar como a aplicação vai funcionar.

> Faça um teste para ver se sua aplicação já está rodando, execute um `docker-compose up` no terminal da aplicação, espere ate aparecer algo como "node index.js", vá no navegador e acesse `localhost:3000`. Se aparecer o "hello word", pode cancelar o processo no terminar e vamos continuar.

## 3º Passo - Configurar o servidor na aws:
Vamos usar a [CLI da aws](https://github.com/aws/aws-cli), então instale corretamente ela de acordo com seu OS. 

> Para verificar se está tudo instalado corretamente, execute um `aws help` no terminal e vê se é reconhecido.

Agora vamos acessar o [console da aws](https://aws.amazon.com/pt/console/), depois de criar ou logar na sua conta execute `aws configure` no terminal.
Irá pedir 4 informações: 
 - AWS Access Key ID -> Obtém no seu usuário, na aba "My Security Credentials", Chaves de acesso e por fim, Criar nova chave de acesso.
 - AWS Secret Access Key -> Vem junto com o Access Key ID.
 - Default region name -> Recomendo por `us-east-1`, pois até onde sei, é gratuito.
 - Default output format -> Testei colocando `yaml`, e funcionou...

## 4º Passo - Criar um novo serviço/servidor na awsec2 com nossa aplicação rodando lá:
Antes, precisamos instalar o [Docker Machine](https://docs.docker.com/machine/install-machine/).

> Para verificar se está tudo instalado corretamente, execute um `docker-machine -h` no terminal e vê se é reconhecido.

Antes de continuar...
> O docker-machine, nós ajuda a criar um servidor "tanto na aws, no digitalOcean, azure... ele suporta vários", tudo isso automaticamente, apenas precisamos informar o serviço que queremos utilizar, como isso, ele vai criar o servidor, instalar o docker, executar rotas... 

Execute então um `docker-machine create --driver amazonec2 nomeincrivel` e esperar a mágica acontecer.
> Se der o erro "Error setting machine configuration from flags provided...": basta executar `export AWS_ACCESS_KEY_ID=suaChaveDeAcesso`, `export AWS_SECRET_ACCESS_KEY=suaChaveSecretaDeAcesso` e `export AWS_DEFAULT_REGION=aRegiaoQueEscolheuNoPasso3`. Execute esses 3 comandos no terminal e tente novamente.

Depois de executar, você pode verificar se deu tudo certo indo na AWS -> Serviços -> EC2 -> Running instances. Se apareceu na lista a máquina que você criou, podemos prosseguir.

## 5º Passo - Configurar nossa aplicação dentro da máquina que criamos:
Para isso execute `docker-machine env nomedaMaquina` no terminal.
Se apareceu os export's, ocorreu tudo bem. No final do retorno do terminal vai ter um código parecido com `eval $(docker...)`, basta você copiar, e executar esse comando para finalizar essa configuração.
A partir de agora, todo comando que você executar do docker, não estará mais na sua máquina, e sim, lá na aws ec2. 

## 6º Passo - Subir nossa aplicação:
Agora vamos subir nosso código, e fazer com que ele fique online executando na aws ec2, para isso é bem simples, basta executar `docker-compose up -d`. Esse comando pode demorar um pouco, pois ele vai construir a sua aplicação la nos servidores.

> Para testar, no console da aws, na mesma aba que estava aparecendo sua máquina, na lateral esquerda do site, vá na opção "Security Groups". Vai ter um grupo chamado "docker-machine", clique nele, vá em "Inbound", "Edit". Add uma nova regra com as seguintes opções: 
 - Custom TCP Rule;
 - Port Range: 3000; "A mesma que escolhemos durante a configuração"
 - Source: Anywhere;
> Salva, volta em "Instances", copie o IPV4 Public IP da sua máquina criada na aws ec2. Cole esse ip no navegador sucedido de `:3000`. Se aparecer o "Hello Word", parabéns, sua aplicação está online ^^.

Made with 💜 by [Guilherme Bafica](https://github.com/guibafica) 👋 