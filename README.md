# tech-blog-web-page-example
Este repositório contém uma página web estática que simula a página inicial de um blog de tecnologia. A página foi criada com o propósito de ser utilizada em um estudo de caso sobre servidores web.

## 1. Créditos

O layout original foi desenvolvido e disponibilizado por [HTML5 UP](https://html5up.net/). Modificamos e utilizamos este layout de acordo com a licença [Creative Commons](https://html5up.net/license).

## 2. Configurações aplicadas:

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
        root   /var/www/tech-blog-web-page-example/src;
        index  index.html index.htm;
    }

    location /images/ {
        sendfile   on;
        tcp_nopush on;
        alias /var/www/tech-blog-web-page-example/src/images/;
    }
```

## 3. Scripts utilizados

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

### Identificação dos picos de CPU e memória (script python)

```python
import pandas as pd

def extract_cpu_rss(file_path):
    with open(file_path, 'r') as file:
        lines = file.readlines()

    prevcpu = ""
    prevrss = ""
    groups = [[]]
    current_group = 0

    i = 0
    while i < len(lines):
        line1 = lines[i].strip()

        if '%CPU' in line1 and "Average:" not in line1:
            time = line1.split()[0]
            cpu = lines[i + 1].strip().split()[7]
            rss = lines[i + 4].strip().split()[6]

            if cpu != prevcpu or rss != prevrss:
                if cpu == "0.00" and groups[current_group]:
                    groups.append([])
                    current_group += 1

                if cpu != "0.00":  # Adicionar amostras onde CPU não é zero
                    groups[current_group].append((time, cpu, rss))
                prevcpu = cpu
                prevrss = rss

        i += 1

    return groups

file_path = 'cenario1caso3.txt'
cpu_rss_groups = extract_cpu_rss(file_path)

# Preparar dados para salvar no Excel
data_for_excel = []

for i, group in enumerate(cpu_rss_groups):
    if group:  # Verifica se o grupo não está vazio
        max_cpu_sample = max(group, key=lambda x: float(x[1]))
        max_rss_sample = max(group, key=lambda x: int(x[2]))
        data_for_excel.append({
            "Grupo": i + 1,
            "Horário Max CPU": max_cpu_sample[0],
            "Valor Max CPU": max_cpu_sample[1],
            "Horário Max RSS": max_rss_sample[0],
            "Valor Max RSS": max_rss_sample[2]
        })

# Criar DataFrame e salvar em Excel
df = pd.DataFrame(data_for_excel)
excel_filename = "cpu_rss_max_values_3.xlsx"
df.to_excel(excel_filename, index=False)

print(f"Arquivo '{excel_filename}' salvo com sucesso.")
```
