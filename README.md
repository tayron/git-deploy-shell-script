# Deploy com git usando Shell Script

Acesse a pasta **/var/www/html** do servidor de produção via ssh e clone seu projeto
```ssh
git clone --branch="staging" --depth 50 https://url-do-projeto-aqui
```ssh

Crie um arquivo ssh com o seguinte conteúdo:
```ssh
#!/bin/bash

dir_www="/var/www/html/"
dir_project="site"

echo "--------------------------------------------------------"
echo "INICIO DO DEPLOY"
echo "--------------------------------------------------------"

echo
echo "1) Removendo backup antigo"
sudo rm -rf "${dir_www}${dir_project}_backup"

echo
echo "2) Realizando novo backup"
sudo cp -rf "${dir_www}${dir_project}" "${dir_www}${dir_project}_backup"

echo
echo "3) Abrindo diretório: ${dir_www}${dir_project}"
cd "${dir_www}${dir_project}"

echo
echo "4) Atualizando repositório"
sudo git checkout staging -f
sudo git pull --depth 1 -f

echo
echo "5) Atualizando dependências"
sudo composer install

echo
echo "6) Atualizando as tabelas do banco"
sudo php bin/cake.php migrations migrate

echo
echo "7) Limpando cache";
echo
sudo ./clearCache.sh

echo "9) Dando permissões nas pastas logs, tmp, android, arquivos e videos"
sudo chmod 777 -R logs/
sudo chmod 777 -R tmp/
sudo chmod 777 -R webroot/upload/

echo
echo 
echo "¯\_(ツ)_/¯ Acabou...."
echo

echo
echo "--------------------------------------------------------"
echo "FIM DO DEPLOY"
echo "--------------------------------------------------------"

```

