# Deploy com Git usando Shell Script

Acesse a pasta **/var/www/html** do servidor de produção via ssh e clone seu projeto
```ssh
git clone --branch="staging" --depth 1 https://url-do-projeto-git-aqui
```

Crie um arquivo ssh chamado: **deploy_staging.sh** com o seguinte conteúdo:
```ssh
#!/bin/bash

VERMELHO='\033[0;31m'
VERDE='\033[0;32m'
AZUL='\033[0;34m'
FIM_COR='\033[0m'

dir_www="/root/projetos/"
dir_project="site.com.br"

printf ${VERDE}
echo " ------------------------------------------------------- "
echo "|                                                       |"
echo "|                INICIO DO DEPLOY                       |"
echo "|                                                       |"
echo " ------------------------------------------------------- "
printf ${FIM_COR}

echo
echo
printf ${VERDE}"1) Removendo backup antigo"${FIM_COR}
echo
printf ${AZUL}
rm -rf "${dir_www}${dir_project}_backup"
printf ${FIM_COR}
echo

echo
printf ${VERDE}"2) Realizando novo backup"${FIM_COR}
echo
printf ${AZUL}
cp -rf "${dir_www}${dir_project}" "${dir_www}${dir_project}_backup"
printf ${FIM_COR}
echo

echo
printf ${VERDE}"3) Abrindo diretório: ${dir_www}${dir_project}"${FIM_COR}
echo
printf ${AZUL}
cd "${dir_www}${dir_project}"
printf ${FIM_COR}
echo

echo
printf ${VERDE}"4) Atualizando repositório"${FIM_COR}
echo
printf ${AZUL}
git reset --hard HEAD
git pull
printf ${FIM_COR}
echo

echo
printf ${VERDE}"5) Copiando arquivo de configurações do backup para dentro do projeto"${FIM_COR}
echo
printf ${AZUL}
cp -rf "${dir_www}${dir_project}_backup/app/config/database.php" "${dir_www}${dir_project}/app/config/database.php"
cp -rf "${dir_www}${dir_project}_backup/app/config/app.php" "${dir_www}${dir_project}/app/config/app.php"
printf ${FIM_COR}
echo

echo
printf ${VERDE}"6) Instalando novas dependências"${FIM_COR}
echo
printf ${AZUL}
docker exec -t container_name composer install
printf ${FIM_COR}
echo

echo
printf ${VERDE}"7) Atualizando as tabelas do banco"
echo
printf ${AZUL}
docker exec -t container_name php artisan migrate
printf ${FIM_COR}
echo

echo
printf ${VERDE}"8) Atualizando serviço agendados: cron"
echo
printf ${AZUL}
docker cp ${dir_www}${dir_project}/build/crontab container_name:/etc/crontab
docker exec -i container_name echo -en '\n\n' >> /etc/crontab
docker exec -i container_name crontab /etc/crontab
docker exec -i container_name crontab -l
docker exec -i container_name service cron status
printf ${FIM_COR}
echo

printf ${VERDE}
echo " ------------------------------------------------------- "
echo "|                                                       |"
echo "|                   FIM DO DEPLOY                       |"
echo "|                                                       |"
echo "|                    ¯\_(ツ)_/¯                         |"
echo "|                                                       |"
echo " ------------------------------------------------------- "
printf ${FIM_COR}


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
