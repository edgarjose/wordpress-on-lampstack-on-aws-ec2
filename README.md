# WordPress numa instância EC2 com LAMP

Laboratório de uma instalação WordPress com Apache, PHP e MySQL/MariaDB na mesma instância. O repositório contém um guia, não uma implementação automatizada.

## Antes de começar

Criar uma instância de laboratório com uma distribuição suportada, Apache, uma versão de PHP compatível com o WordPress atual e MySQL/MariaDB. Instalar os pacotes a partir dos repositórios oficiais da distribuição e confirmar os serviços ativos.

Os comandos históricos deste repositório usavam Amazon Linux 2 e PHP 7.4. Não reproduzir esses runtimes antigos numa instalação nova. Consultar os [requisitos atuais do WordPress](https://wordpress.org/about/requirements/) e a documentação da distribuição.

Usar acesso administrativo restrito e não publicar MySQL na internet. Este desenho de uma única instância não é uma arquitetura de alta disponibilidade.

## Criar a base de dados

Na consola administrativa do MySQL/MariaDB, escolher uma palavra-passe forte e única, diferente de qualquer exemplo publicado:

```sql
CREATE DATABASE wordpressdb CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
CREATE USER 'wordpress_app'@'localhost' IDENTIFIED BY 'SUBSTITUIR_POR_UM_SEGREDO_FORTE';
GRANT ALL PRIVILEGES ON wordpressdb.* TO 'wordpress_app'@'localhost';
```

Não executar o placeholder literalmente nem colocar credenciais no Git. Confirmar que o utilizador consegue ligar-se à base de dados local.

## Obter e configurar WordPress

Executar os comandos na instância; requer `curl`, `tar` e permissões administrativas:

```bash
LAB_DIR="$(mktemp -d)"
curl -fsSL https://wordpress.org/latest.tar.gz -o "$LAB_DIR/wordpress.tar.gz"
tar -xzf "$LAB_DIR/wordpress.tar.gz" -C "$LAB_DIR"
cp "$LAB_DIR/wordpress/wp-config-sample.php" "$LAB_DIR/wordpress/wp-config.php"
```

Editar esse `wp-config.php` com `DB_NAME=wordpressdb`, `DB_USER=wordpress_app`, a palavra-passe escolhida e `DB_HOST=localhost`. Gerar salts próprios através do [serviço oficial do WordPress](https://api.wordpress.org/secret-key/1.1/salt/).

Depois de configurar o ficheiro, copiar o conteúdo a partir do diretório correto:

```bash
sudo cp -a "$LAB_DIR/wordpress/." /var/www/html/
```

Aplicar permissões conforme o utilizador real do Apache na distribuição. Manter `wp-config.php` protegido e conceder escrita apenas aos diretórios que o WordPress necessitar. O utilizador do servidor não deve ter permissões excessivas sobre todos os ficheiros por conveniência.

## Verificar

Abrir o endereço do servidor e concluir o instalador, usando um utilizador administrador diferente de exemplos comuns. Testar login, um artigo e o upload de media. Consultar os logs PHP/Apache se houver erros.

Configurar HTTPS antes de usar credenciais num site acessível publicamente. Planear atualizações e backups dos ficheiros e da base de dados.

## Custos e limpeza

EC2, volumes EBS e IPv4 públicos podem gerar custos. Após o ensaio, terminar os recursos dedicados e rever volumes/Elastic IPs. Se o objetivo for manter o site, definir backups e testar o restauro em vez de eliminar recursos.

[Guia de migração entre EC2 e RDS](https://github.com/edgarjose/migrar-monolitico-para-microservicos-na-aws)
