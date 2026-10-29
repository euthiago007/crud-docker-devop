# Trabalho 2 — Deploy Automático em VM

**Aluno:** Thiago Duarte

---

## 1. Objetivo

O segundo trabalho consiste em automatizar a atualização da aplicação CRUD em uma máquina virtual por meio do GitHub Actions e de um Runner local (*self-hosted*).

A aplicação utilizada é a mesma documentada anteriormente, executada em containers Docker no Ubuntu Server instalado na Oracle VirtualBox.

**Responsável pela configuração do deploy:** Thiago Duarte.

## 2. Ambiente utilizado

| Tecnologia | Finalidade |
|---|---|
| Ubuntu Server | Sistema operacional da máquina virtual. |
| Oracle VirtualBox | Virtualização do ambiente. |
| Docker e Docker Compose | Execução e gerenciamento dos containers. |
| Git e GitHub | Controle de versão e hospedagem do projeto. |
| GitHub Actions | Automação do processo de deploy. |
| GitHub Actions Runner (*self-hosted*) | Execução dos workflows diretamente na máquina virtual. |

## 3. Funcionamento do Runner local

O Runner local foi instalado e registrado no repositório GitHub, sendo configurado como um serviço do Ubuntu Server.

Ele permite que os workflows do GitHub Actions sejam executados diretamente na máquina virtual que hospeda a aplicação.

Para executar o deploy, a máquina virtual precisa estar ligada, conectada à internet e com o Runner em execução.

## 4. Workflow de deploy

O workflow está localizado em `.github/workflows/deploy.yml`.

Ele possui dois gatilhos:

- **Push:** inicia automaticamente o workflow quando alterações são enviadas para a branch `main`.
- **Execução manual:** permite iniciar o workflow pela aba **Actions** do GitHub, utilizando a opção **Run workflow**.

### Etapas do processo

1. **Atualização do código:** o Runner acessa o diretório do projeto na VM e executa `git pull origin main` para obter as alterações mais recentes.
2. **Remoção da versão anterior:** o comando `docker compose down` encerra e remove os containers da versão anterior.
3. **Inicialização da versão atualizada:** o comando `docker compose up -d --build` inicia novamente os serviços com a aplicação atualizada.

Dessa forma, após um `push` para a branch `main`, o GitHub Actions pode executar o processo de atualização sem que seja necessário iniciar o deploy manualmente.

### Detalhamento das etapas de deploy

**Descendo a versão anterior**

Essa etapa acessa o diretório do projeto na máquina virtual por meio do comando `cd`, garantindo que os comandos Docker sejam executados no local correto. Em seguida, `docker compose down` encerra e remove os containers e a rede criados pelo Docker Compose, preparando o ambiente para iniciar a versão atualizada.

**Subindo versão mais nova**

Após a remoção dos containers anteriores, essa etapa acessa novamente o diretório do projeto e executa `docker compose up -d --build`. O comando inicia os serviços definidos no arquivo `docker-compose.yml`. A opção `-d` permite que os containers sejam executados em segundo plano, enquanto `--build` solicita a construção das imagens antes da inicialização, quando houver configurações de build.

**Fluxo entre as etapas**

Primeiro, o workflow encerra a execução dos containers anteriores para preparar o ambiente. Em seguida, inicia novamente os serviços utilizando a versão mais recente do projeto, que já foi obtida na etapa de atualização do código (`git pull origin main`). Dessa forma, o deploy substitui a execução anterior pela nova versão da aplicação.

## 5. Acesso à aplicação

Com a máquina virtual ligada e os containers em execução, a aplicação pode ser acessada no computador hospedeiro pelos seguintes endereços:

| Serviço | Endereço |
|---|---|
| CRUD | http://localhost:8080 |
| phpMyAdmin | http://localhost:8081 |

O acesso SSH à máquina virtual utiliza a porta `22`, configurada por meio do encaminhamento de portas da rede NAT da VirtualBox.

Os endereços `localhost` são válidos no computador hospedeiro. Para acessar a aplicação a partir de outro computador, é necessário utilizar o endereço IP apropriado e possuir uma configuração de rede que permita o acesso.

## 6. Persistência e reinicialização

O banco de dados utiliza o volume Docker `mysql_data`, preservando os registros durante a recriação dos containers.

Os serviços também possuem a política de reinicialização `restart: unless-stopped`, permitindo que os containers voltem a iniciar após a reinicialização da máquina virtual, desde que não tenham sido parados manualmente.

## 7. Demonstração do deploy automático

Para demonstrar o funcionamento do deploy automático:

1. Realizar uma pequena alteração em algum arquivo da aplicação.
2. Enviar a alteração para a branch `main` do repositório GitHub.
3. Acessar a aba **Actions** e verificar a execução do workflow pelo Runner local.
4. Confirmar que as etapas de atualização, remoção dos containers anteriores e inicialização da nova versão foram concluídas com sucesso.
5. Acessar http://localhost:8080 e verificar a alteração no navegador.

Esse procedimento demonstra a integração entre GitHub, GitHub Actions, Runner local e Docker Compose para automatizar a implantação de alterações na aplicação.

## 8. Considerações finais

O projeto demonstra como o GitHub Actions pode ser integrado a um Runner local para automatizar a implantação de aplicações em uma máquina virtual.

A utilização do Docker Compose permite gerenciar os serviços da
