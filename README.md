# tech-blog-web-page-example
Este repositório contém uma página web estática que simula a página inicial de um blog de tecnologia. A página foi criada com o propósito de ser utilizada em um estudo de caso sobre servidores web.

## Créditos

O layout original foi desenvolvido e disponibilizado por [HTML5 UP](https://html5up.net/). Modificamos e utilizamos este layout de acordo com a licença [Creative Commons](https://html5up.net/license).

## Configurações aplicadas:

### Cenário 1

```bash
    sendfile   off;
    gzip       off;
```

### Cenário 2

```bash
    sendfile   off;
    gzip        on;
    gzip_types
      application/javascript
      text/javascript
      text/css;
```

### Cenário 3

```bash
    sendfile   on;
    tcp_nopush on;
    gzip      off;
```

### Cenário 4

```bash
    location / {
        gzip  on;
        gzip_types
          application/javascript
          text/javascript
          text/css;
        root   /var/www/html/tech-blog-web-page-example/src;
        index  index.html index.htm;
    }

    location /images/ {
        sendfile   on;
        tcp_nopush on;
        alias /var/www/html/tech-blog-web-page-example/src/images/;
    }
```

## Scripts utilizados

### Execução dos casos de teste do lado do cliente via ferramenta ApacheBench (ab)

#### Caso 1

```bash
ab -n 10000 -c 500 http://ec2-3-22-114-17.us-east-2.compute.amazonaws.com/
```

#### Caso 2

```bash
ab -n 20000 -c 1000 http://ec2-3-22-114-17.us-east-2.compute.amazonaws.com/
```

### Monitoramento de CPU e memória via ferramenta pidstat

```bash
pidstat -p $(ps aux | grep "nginx: worker process" | grep -v "grep" | awk '{print $2}') -u -r 1 > cen1amo1.txt
```

### Obtenção dos picos de CPU e memória do monitoramento feito

#### CPU

```bash
grep -v 'UID' cen1amo1.txt | awk '{print $8}' | sort -nr | head -n 1
```

#### Memória

```bash
grep -v 'UID' cen1amo1.txt | awk '{print $7}' | sort -nr | head -n 1
```