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

    name: Build e Deploy  #Nomeia

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

---
