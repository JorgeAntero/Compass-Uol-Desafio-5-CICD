# - 🔒 Projeto de Bolsas DevSecOps/AWS,  Compass UOL, abril 2025 🔒 -

## 🐈‍⬛ CI/CD com Git Actions 🐈‍⬛

## 📜 0 - Breve resumo >
Para o quinto projeto nos foi proposto realizar a automatização do ciclo completo de desenvolvimento, build, deploy e execução de uma FastAPI, utiliando Github Actions, Docker Hub, e ArgoCD com Kubernetes.  

---
## 🗳️ 1 - Criação dos repositórios >
### Primeiramente, criei os dois repositórios necessários:  

![Primeiro print](/Prints/1.1.png)  
`Primeiro criei o repositório pro FastAPI.`  
[Link para o repositório](https://github.com/JorgeAntero/Projeto-5-My-App)  

![Segundo print](/Prints/1.2.png)  
`E depois criei o repositório dos manifestos.`  
[Link para o repositório](https://github.com/JorgeAntero/Projeto-5-Manifests)  

---
## 💨 2 - Conteúdo  do MyApp >
### Em seguida, criei os conteúdos presentes no MyApp, como a FastAPI, arquivo requirements, Dockerfile e o arquivo yaml do CICD:  

![Terceiro print](/Prints/2.1.png)  
`Utilizei o arquivo já disponibilizado para criar a FastAPI.`  

![Quarto print](/Prints/2.2.png)  
`Especifica o que é necessário para a FastAPI funcionar corretamente.`  

![Quinto print](/Prints/2.3.png)  
`É uma dockerfile comum. É interessante destacar a última linha, que inicia o servidor Uvicorn.`  

### Também criei o diretório "/.github/workflows" e o arquivo "ci-cd.yaml" dentro dele:  

    name: Build e Deploy  # Nomeia

    on:
    push:
        branches:
        - main
    pull_request:
        branches:
        - main
    # As linhas acima servem para criar os gatilhos para quando um push ou um pull request forem executados na main.

    jobs:
    build-push:
        runs-on: ubuntu-latest

        steps:
        - name: Checar código
            uses: actions/checkout@v3
        # Clona o repo atual para o runner.

        - name: Login no Docker Hub
            uses: docker/login-action@v2
            with:
            username: ${{ secrets.DOCKER_USERNAME }}
            password: ${{ secrets.DOCKER_PASSWORD }}
        # Faz o login no Docker Hub utilizando os segredos (ainda mostrarei como fiz).

        - name: Buildar imagem Docker
            run: |
            docker build -t ${{ secrets.DOCKER_USERNAME }}/hello-app:latest .

        - name: Push na imagem Docker
            run: |
            docker push ${{ secrets.DOCKER_USERNAME }}/hello-app:latest

        - name: Checkout no repo manifests
            uses: actions/checkout@v3
            with:
            repository: JorgeAntero/Projeto-5-Manifests
            token: ${{ secrets.GITHUB_TOKEN }}
            path: root
        # Clona o repositório dos manifestos na root (também mostrarei melhor em breve)

        - name: Atualiza a tag de imagem no manifests
            run: |
            sed -i 's|image: .*|image: ${{ secrets.DOCKER_USERNAME }}/hello-app:latest|' root/root/deployment.yaml

        - name: Commit e Push para o repo manifests
            run: |
            cd root/root
            git config user.name "github-actions"
            git config user.email "actions@github.com"
            
            if git diff --quiet; then
                echo "Sem diferenças para commitar."
            else
                git add deployment.yaml
                git commit -m "Atualizando taga para latest"
                git push origin main
            fi
        # Verifica alterações, além de configurar e-mail para comit.

---
## 🔑 3 - Configurando os segredos >
### Para o terceiro passo, configurei os segredos necessários para o funcionamento do nosso workflow:  

![Sexto print](/Prints/3.1.png)   
`Primeiro executei o comando "ssh-keygen -t ed25519 -C "github-actions" -f argo-key" no meu terminal, para criar as chaves ssh pública e privada.`  

![Sétimo print](/Prints/3.2.png)  
`Depois, no repositório My-App, criei os 3 segredos:`
>- DOCKER_USERNAME: Para meu username do Docker Hub:  
>- DOCKER_PASSWORD: Para minha senha do Docker Hub;  
>- SSH_PRIVATE_KEY: Para o conteúdo da chave privada gerada anteriormente;  

![Oitavo print](/Prints/3.3.png)  
`Por fim, criei no repositório dos manifestos uma Deploy Key com o conteúdo da chave pública gerada anteriormente, dando a premissão de leitura e escrita para ela.`


---
## 📓 4 - Conteúdo do repositório de manifestos >
### Primeiro, criei o diretório "/root" e adicionei dois arquivos yaml nele:  

![Nono print](/Prints/4.1.png)  
`O primeiro foi o arquivo deployment.yaml, que descreve como a aplicação deve rodar no cluster kubernetes.` 

![Décimo print](/Prints/4.2.png)  
`E o segundo foi o arquivo service.yaml, que expõe os pods do Deployment para usuários externos.`  

---
## 🦑 5 - Rodando no ArgoCD >
### Enfim, precisei rodar a aplicação no argo, pra isso, segui a etapa de configuração presente no [projeto passado](https://github.com/JorgeAntero/Compass-Uol-Desafio-4-GITops) - etapa 4. Depois disso:  

![Print Onze](/Prints/5.1.png)  
`Após identificar o nome do serviço, hospedei ele na porta 8080`
  
![Print Doze](/Prints/5.2.png)  
`Ao acessar o link, o funcionamento estava correto!`

---
## 🫸 6 - Testando Pull Request >
### O último passo foi testar o pull request e atualização de mensagem da aplicação: 


![Print Treze](/Prints/6.1.png)  
`Primeiro, troquei a mensagem do "main.py".`  

![Print Quatorze](/Prints/6.2.png)  
`Depois realizei o Pull Request.`

![Print Quinze](/Prints/6.3.png)  
`A mensagem acima do git Action demonstra o sucesso.`
 
![Print Dezesseis](/Prints/6.4.png)  
`Ao acessar o link novamente, vemos que a mensagem atualizou, indicando o sucesso do projeto!`
