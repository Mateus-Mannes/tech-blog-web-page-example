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

```bash
#!/bin/bash

# Verifica se o número correto de argumentos foi fornecido
if [ "$#" -ne 5 ]; then
    echo "Uso: $0 <URL> <Total de Requisições> <Concorrência> <Número de Testes> <Arquivo de Log>"
    exit 1
fi

# Configurações a partir dos argumentos
URL=$1                          # URL para teste
TOTAL_REQUESTS=$2               # Número total de requisições por teste
CONCURRENCY=$3                  # Número de requisições concorrentes
NUM_TESTS=$4                    # Número de vezes que o teste será executado
LOG_FILE=$5                     # Arquivo de log para resultados

# Verifica se o arquivo de log já existe e o remove para iniciar um novo
if [ -f "$LOG_FILE" ]; then
    rm "$LOG_FILE"
fi

# Executa os testes
for i in $(seq 1 $NUM_TESTS); do
    echo "Teste $i de $NUM_TESTS..."

    # Executa o Apache Bench
    result=$(ab -n $TOTAL_REQUESTS -c $CONCURRENCY $URL | grep 'Requests per second')

    # Extrai e salva o número de requisições por segundo
    echo "$result" >> $LOG_FILE

    # Espera por 3 segundos antes da próxima execução
    sleep 3
done

echo "Testes completados. Resultados salvos em $LOG_FILE."
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