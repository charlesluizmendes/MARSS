# MARSS-RISCV Simulador

Para a instalação e simulação do mesmo foi utilizado o  Ubuntu 22.04 LTS.

## Ambiente Linux

Instalar os pacotes para a utilizacao do marss-riscv:

```
$ sudo apt-get update
$ sudo apt-get install build-essential
$ sudo apt-get install libssl-dev
$ sudo apt-get install libsdl1.2-dev
$ sudo apt-get install libcurl4-openssl-dev
```

## Baixando o código-fonte:

Baixe o projeto no diretorio de sua preferencia:

```
$ git clone https://github.com/Lab-COMPASSO/marss-riscv.git
```

Acesse a pasta "src" do marss-riscv:

```
$ cd marss-riscv/src/
```

Compile o marss-riscv:

```
$ make
```

Acesse o diretorio "configs" do simulador marss-riscv:

```
$ cd ../configs
```

## Baixando imagem do simulador:

Apos isso execute os seguintes comandos para baixar a imagem pré-compiladas de espaço do usuário, bootloader e kernel RISC-V de 32 e 64 bits:

```
$ wget https://cs.binghamton.edu/~marss-riscv/marss-riscv-images.tar.gz
$ tar -xvzf marss-riscv-images.tar.gz
$ cd marss-riscv-images/riscv64-unknown-linux-gnu/
$ xz -d -k -T 0 riscv64.img.xz
```

Agora podemos voltar para a pasta "src" do simulador marss-riscv:

```
$ cd ../../../src
```

## Copiando os arquivos de configurações:

Copie e substitua os arquivos de configuracoes [riscv64_inorder_soc.cfg](https://github.com/charlesluizmendes/MARSS/blob/main/configs/riscv64_inorder_soc.cfg) e [riscv64_outoforder_soc.cfg](https://github.com/charlesluizmendes/MARSS/blob/main/configs/riscv64_outoforder_soc.cfg) para a pasta "configs" do marss-riscv

## Executando o simulador:

Para cada modo de execução, no modo In Order e Out of Order, execute os [benchmarks](#executando-benchmarks).

### Executando no modo In Order:

Execute o seguinte comando para executar o emulador marss-riscv no modo In Order:

```
$ ./marss-riscv ../configs/riscv64_inorder_soc.cfg
```

### Executando no modo Out of Order:

Execute o seguinte comando para executar o emulador marss-riscv no modo Out of Order:

```
$ ./marss-riscv ../configs/riscv64_outoforder_soc.cfg
```

## Configurando o simulador:

### Configurando a data/hora:

Ao executar o simulador marss-riscv, primeiramente ajuste a data e hora do sistema:

```
$ rc-service ntpd stop
$ date -s "yyyy-MM-dd hh:mm:ss"
```

### Baixando o CoreMark:

Para executar os banchmarks no simulador marss-riscv, precisamos baixar o CoreMark e compilar o mesmo:

```
$ git clone https://github.com/eembc/coremark.git
$ cd coremark/
$ make
$ cd ..
```

### Baixando o utilitário do simulador:

Para executar os comandos do banchmarks no simulador marss-riscv, precisamos baixar e compilar o mesmo:

```
$ git clone https://github.com/bucaps/marss-riscv-utils.git
$ cd marss-riscv-utils/
$ make
$ cd ..
```

## Executando Benchmarks

O CoreMark é um benchmarks composto por diferentes kernels computacionais. Ele não mede apenas uma operação isolada, mas combina listas encadeadas, operações de matriz, máquinas de estado finito e cálculos de CRC. Esses componentes exercitam diferentes partes do processador, como acesso à memória, execução aritmética, controle de fluxo e validação de integridade dos resultados.

As três execuções seguintes não representam benchmarks diferentes, mas sim diferentes cenários de entrada do mesmo benchmarks CoreMark. Em todas elas, o CoreMark executa os mesmos kernels internos: lista encadeada, operações de matriz, máquina de estados finitos e CRC.

### Seeds 0x0 0x0 0x66:

```
simstart; \
  ./coremark/coremark.exe 0x0 0x0 0x66 0 7 1 2000 > run_ec1.log; \
simstop;
```

* Seeds 0x0/0x0: definem valores iniciais dos elementos da lista (zeros), garantindo que a validação CRC na saída case com o valor de referência.
* Seed matrix 0x66: afeta valores iniciais usados na multiplicação de matrizes, mas não altera a estrutura do código.
* “0 7 1 2000” (os demais quatro parâmetros):
* 0: modo clássico de FSM (sem loops extras).
* 7: profundidade de 7 para o laço principal de iterações.
* 1: habilita contagem exata de tempo real.
* 2000: tamanho do bloco de trabalho, garantindo que cada iteração passe por operações não
trivialmente pequenas.

### Seeds 0x3415 0x3415 0x66:

```
simstart; \
  ./coremark/coremark.exe 0x3415 0x3415 0x66 0 7 1 2000 > run_ec2.log; \
simstop;
```

* Seeds alteradas: elementos da lista e valores iniciais da FSM agora começam em 0x3415, inserindo um padrão de dados não trivial.
* Manter o mesmo laço de multiplicação de matrizes e parâmetros de iteração.

### Seeds 8 8 8:

```
simstart; \
  ./coremark/coremark.exe 8 8 8 0 7 1 2000 > run_ec3.log; \
simstop;
```

* Seeds incomuns: dado o valor baixo (8), alterações no padrão de acesso de lista e FSM são mínimas, mas suficientes para invalidar o CRC de referência.
