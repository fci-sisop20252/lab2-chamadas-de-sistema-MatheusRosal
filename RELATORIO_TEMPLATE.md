# 📝 Relatório do Laboratório 2 - Chamadas de Sistema

---

## 1️⃣ Exercício 1a - Observação printf() vs 1b - write()

### 💻 Comandos executados:
```bash
strace -e write ./ex1a_printf
strace -e write ./ex1b_write
```

### 🔍 Análise

**1. Quantas syscalls write() cada programa gerou?**
- ex1a_printf: 8 syscalls
- ex1b_write: 7 syscalls
- ex2_leitura: 6 syscalss
- ex3_contador: 42 syscalls
- ex4_copia: 33 syscalls

**2. Por que há diferença entre os dois métodos? Consulte o docs/printf_vs_write.md**
Há diferença, pois o printf é capaz de escrever textos para o usuário, formatar dados e é utilizado para casos que não seja necessário uma performance crítica.
E o write é capaz de escrever, além de texto, dados binários, comtrolar quando enviar os dados e possui um comportamento previsível.

**3. Qual método é mais previsível? Por quê você acha isso?**

```
[O write é mais previsível, pois sempre executa exatamente a quantidade de bytes informada, sem atrasos ou divisão inesperada. Já o printf pode agrupar saídas ou quebrá-las em várias chamadas dependendo do estado do buffer.]
```

---

## 2️⃣ Exercício 2 - Leitura de Arquivo

### 📊 Resultados da execução:
- File descriptor: 3
- Bytes lidos: 127

### 🔧 Comando strace:
```bash
strace -e openat,read,close ./ex2_leitura
```

### 🔍 Análise

**1. Qual file descriptor foi usado? Por que não começou em 0, 1 ou 2?**

```
[Foi usado o 3, porque 0, 1 e 2 já estão reservados para stdin, stdout e stderr. Assim, o próximo descritor livre foi o 3.]
```

**2. Como você sabe que o arquivo foi lido completamente?**

```
[Porque o read() retornou exatamente o tamanho do arquivo (127 bytes) e em seguida retornou 0, indicando fim de arquivo.]
```

**3. Por que verificar retorno de cada syscall?**

```
[Porque falhas podem ocorrer (arquivo inexistente, permissões insuficientes, disco corrompido). Só assim garantimos que a operação foi realmente bem-sucedida.]
```

---

## 3️⃣ Exercício 3 - Contador com Loop

### 📋 Resultados (BUFFER_SIZE = 64):
- Linhas: 25 (esperado: 25)
- Caracteres: 1300
- Chamadas read(): 21
- Tempo: 0.000459 segundos

### 🧪 Experimentos com buffer:

| Buffer Size | Chamadas read() | Tempo (s) |
|-------------|-----------------|-----------|
| 16          |    82           |  0.0012   |
| 64          |    21           |  0.000459 |
| 256         |     6           |  0.00018  |
| 1024        |     2           |  0.00009  |

### 🔍 Análise

**1. Como o tamanho do buffer afeta o número de syscalls?**

```
[Quanto menor o buffer, mais read() precisam ser chamadas para percorrer o arquivo. Buffers maiores reduzem a quantidade de syscalls.]
```

**2. Todas as chamadas read() retornaram BUFFER_SIZE bytes? Discorra brevemente sobre**

```
[Não. A maioria retorna exatamente BUFFER_SIZE, mas a última chamada pode retornar menos, pois o arquivo acaba. Isso é esperado e indica o fim do arquivo.]
```

**3. Qual é a relação entre syscalls e performance?**

```
[SCada syscall tem custo de transição usuário → kernel. Menos chamadas significam menos overhead, logo, melhor performance.]
```

---

## 4️⃣ Exercício 4 - Cópia de Arquivo

### 📈 Resultados:
- Bytes copiados: 1364
- Operações: 6
- Tempo: 0.000642 segundos
- Throughput: 2124 KB/s

### ✅ Verificação:
```bash
diff dados/origem.txt dados/destino.txt
```
Resultado: [X] Idênticos [ ] Diferentes

### 🔍 Análise

**1. Por que devemos verificar que bytes_escritos == bytes_lidos?**

```
[Para garantir que não houve perda de dados durante a cópia.]
```

**2. Que flags são essenciais no open() do destino?**

```
[O_WRONLY | O_CREAT | O_TRUNC -> permitem escrita, criam o arquivo se não existir e sobrescrevem se já existir.]
```

**3. O número de reads e writes é igual? Por quê?**

```
[Sim, cada leitura gera uma escrita correspondente. Porém, strace mostra escritas extras de printf, que não fazem parte da cópia em si.]
```

**4. Como você saberia se o disco ficou cheio?**

```
[O write() retornaria menos bytes do que o solicitado, indicando falha.]
```

**5. O que acontece se esquecer de fechar os arquivos?**

```
[O SO pode manter buffers não liberados, resultando em perda de dados ou arquivos corrompidos. Além disso, há risco de esgotar descritores.]
```

---

## 🎯 Análise Geral

### 📖 Conceitos Fundamentais

**1. Como as syscalls demonstram a transição usuário → kernel?**

```
[Cada syscall (open, read, write, close) interrompe o programa em modo usuário e delega a tarefa ao kernel, que acessa recursos protegidos do sistema.]
```

**2. Qual é o seu entendimento sobre a importância dos file descriptors?**

```
[São a referência fundamental para que o kernel saiba qual recurso (arquivo, socket, terminal) está sendo manipulado.]
```

**3. Discorra sobre a relação entre o tamanho do buffer e performance:**

```
[Buffers maiores reduzem a quantidade de syscalls necessárias e melhoram a performance, mas aumentam uso de memória. Buffers muito pequenos aumentam overhead.]
```

### ⚡ Comparação de Performance

```bash
# Teste seu programa vs cp do sistema
time ./ex4_copia
time cp dados/origem.txt dados/destino_cp.txt
```

**Qual foi mais rápido?** time ./ex4_copia

**Por que você acha que foi mais rápido?**

```
[O arquivo copiado era pequeno. Nesse caso, o comando `cp` realiza verificações adicionais de permissões, atributos e metadados e isso torna sua execução um pouco mais lenta.
]
```

---

## 📤 Entrega
Certifique-se de ter:
- [ ] Todos os códigos com TODOs completados
- [ ] Traces salvos em `traces/`
- [ ] Este relatório preenchido como `RELATORIO.md`

```bash
strace -e write -o traces/ex1a_trace.txt ./ex1a_printf
strace -e write -o traces/ex1b_trace.txt ./ex1b_write
strace -o traces/ex2_trace.txt ./ex2_leitura
strace -c -o traces/ex3_stats.txt ./ex3_contador
strace -o traces/ex4_trace.txt ./ex4_copia
```
# Bom trabalho!
