# Deploy com Git usando Shell Script

Acesse a pasta **/var/www/html** do servidor de produção via ssh e clone seu projeto
```ssh
git clone --branch="staging" --depth 1 https://url-do-projeto-git-aqui
```

Crie um arquivo ssh chamado: **deploy_staging.sh** com o seguinte conteúdo:
```ssh
#!/bin/bash

dir_www="/var/www/html/"
dir_project="site"

echo " ------------------------------------------------------- "
echo "|                                                       |"
echo "|                INICIO DO DEPLOY                       |"
echo "|                                                       |"
echo " ------------------------------------------------------- "
echo
echo

echo "1) Removendo backup antigo"
echo
sudo rm -rf "${dir_www}${dir_project}_backup"
echo

echo "2) Realizando novo backup"
echo
sudo cp -rf "${dir_www}${dir_project}" "${dir_www}${dir_project}_backup"
echo

echo "3) Abrindo diretório: ${dir_www}${dir_project}"
echo
cd "${dir_www}${dir_project}"
echo

echo "4) Atualizando repositório"
echo
sudo git reset --hard HEAD
sudo git pull > ~/deploy_staging.log
echo

echo "5) Copiando arquivo de configurações do backup para dentro do projeto"
echo
sudo cp -rf "${dir_www}${dir_project}_backup/config/Environment/production.env" "${dir_www}${dir_project}/config/Environment/production.env"
echo

echo "6) Atualizando dependências"
echo
sudo composer install
echo

echo "7) Atualizando as tabelas do banco"
echo
sudo php bin/cake.php migrations migrate
echo

echo "8) Limpando cache";
echo
sudo ./clearCache.sh
echo

echo "9) Dando permissões em todos os arquivos"
echo
sudo chmod 755 *
echo

echo "10) Dando permissões nas pastas logs, tmp, uploads"
echo
sudo chmod 777 -R logs/
sudo chmod 777 -R tmp/
sudo chmod 777 -R webroot/uploads/
echo

echo "11) Removendo permissão dos arquivos Shell Script"
echo
sudo chmod 760 *.sh
echo
echo

echo " ------------------------------------------------------- "
echo "|                                                       |"
echo "|                   FIM DO DEPLOY                       |"
echo "|                                                       |"
echo "|                    ¯\_(ツ)_/¯                         |"
echo "|                                                       |"
echo " ------------------------------------------------------- "

```
Crie um arquivo ssh chamado: **deploy_revert.sh** com o seguinte conteúdo:
```ssh
#!/bin/bash

dir_www="/var/www/html/"
dir_project="site"

echo " ------------------------------------------------------- "
echo "|                                                       |"
echo "|                INICIO DO REVERSAO                     |"
echo "|                                                       |"
echo " ------------------------------------------------------- "
echo
echo

echo "1) Removendo deploy atual"
sudo rm -rf "${dir_www}${dir_project}"
echo

echo "2) Restaurando backup"
echo
sudo cp -rf "${dir_www}${dir_project}_backup" "${dir_www}${dir_project}"
echo

echo "3) Abrindo diretório: ${dir_www}${dir_project}"
cd "${dir_www}${dir_project}"

echo "4) Limpando cache";
echo
sudo ./clearCache.sh
echo

echo "5) Dando permissões em todos os arquivos"
echo
sudo chmod 755 *
echo

echo "6) Dando permissões nas pastas logs, tmp e uploads"
echo
sudo chmod 777 -R logs/
sudo chmod 777 -R tmp/
sudo chmod 777 -R webroot/uploads/
echo

echo "7) Removendo permissão dos arquivos Shell Script"
echo
sudo chmod 760 *.sh
echo

echo " ------------------------------------------------------- "
echo "|                                                       |"
echo "|                  FIM DA REVERSÃO                      |"
echo "|                                                       |"
echo "|                    ¯\_(ツ)_/¯                         |"
echo "|                                                       |"
echo " ------------------------------------------------------- "

```

## Utilização dos scripts

###Exemplo de uso para efetuar deploy:

```sh
ssh root@seusite.com.br "./deploy_staging.sh"

```
### Exemplo de uso para reverter o último deploy:

```sh
ssh root@seusite.com.br "./deploy_revert.sh"
```
