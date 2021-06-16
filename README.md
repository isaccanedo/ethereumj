## Ethereum

# 1. Introdução
Neste artigo, damos uma olhada na biblioteca EthereumJ que nos permite interagir com a blockchain Ethereum, usando Java.

Em primeiro lugar, vamos mergulhar brevemente no que esta tecnologia trata.

# 2. Sobre Ethereum
Ethereum é uma criptomoeda que aproveita um banco de dados ponto a ponto distribuído na forma de um blockchain programável, o Ethereum Virtual Machine (EVM). É sincronizado e operado por meio de nós distintos, mas conectados.

A partir de 2017, os Nodes sincronizam o blockchain por consenso, criam moedas por mineração (prova de trabalho), verificam transações, executam contratos inteligentes escritos no Solidity e executam o EVM.

O blockchain é dividido em blocos que contêm estados de conta (incluindo transações entre contas) e prova de trabalho.

# 3. A fachada Ethereum
A classe org.ethereum.facade.Ethereum abstrai e une muitos pacotes do EthereumJ em uma interface fácil de usar.

É possível se conectar a um nó para sincronizar com a rede geral e, uma vez conectado, podemos trabalhar com o blockchain.

Criar um objeto de fachada é tão fácil quanto:

Ethereum ethereum = EthereumFactory.createEthereum();

# 4. Conectando-se à rede Ethereum
Para se conectar à rede, devemos primeiro conectar a um nó, ou seja, um servidor executando o cliente oficial. Os nós são representados pela classe org.ethereum.net.rlpx.Node.

O org.ethereum.listener.EthereumListenerAdapter lida com eventos de blockchain detectados por nosso cliente após a conexão a um nó ter sido estabelecida com sucesso.

### 4.1. Conectando-se à Rede Ethereum
Vamos nos conectar a um nó da rede. Isso pode ser feito manualmente:

```
String ip = "http://localhost";
int port = 8345;
String nodeId = "a4de274d3a159e10c2c9a68c326511236381b84c9ec...";

ethereum.connect(ip, port, nodeId);
```

A conexão com a rede também pode ser feita automaticamente usando um bean:

```
public class EthBean {
    private Ethereum ethereum;

    public void start() {
        ethereum = EthereumFactory.createEthereum();
        ethereum.addListener(new EthListener(ethereum));
    }

    public Block getBestBlock() {
        return ethereum.getBlockchain().getBestBlock();
    }

    public BigInteger getTotalDifficulty() {
        return ethereum.getBlockchain().getTotalDifficulty();
    }
}
```

Podemos então injetar nosso EthBean em nossa configuração de aplicativo. Em seguida, ele se conecta automaticamente à rede Ethereum e começa a baixar o blockchain.

Na verdade, a maior parte do processamento de conexão é convenientemente empacotado e abstraído simplesmente adicionando uma instância org.ethereum.listener.EthereumListenerAdapter à nossa instância org.ethereum.facade.Ethereum criada, como fizemos em nosso método start() acima:

```
EthBean eBean = new EthBean();
Executors.newSingleThreadExecutor().submit(eBean::start);
```

### 4.2. Lidando com Blockchain usando um Listener
Também podemos criar uma subclasse de EthereumListenerAdapter para manipular eventos de blockchain detectados por nosso cliente.

Para realizar esta etapa, precisaremos tornar nosso ouvinte de subclasse:

```
public class EthListener extends EthereumListenerAdapter {
    
    private void out(String t) {
        l.info(t);
    }

    //...

    @Override
    public void onBlock(Block block, List receipts) {
        if (syncDone) {
            out("Net hash rate: " + calcNetHashRate(block));
            out("Block difficulty: " + block.getDifficultyBI().toString());
            out("Block transactions: " + block.getTransactionsList().toString());
            out("Best block (last block): " + ethereum
              .getBlockchain()
              .getBestBlock().toString());
            out("Total difficulty: " + ethereum
              .getBlockchain()
              .getTotalDifficulty().toString());
        }
    }

    @Override
    public void onSyncDone(SyncState state) {
        out("onSyncDone " + state);
        if (!syncDone) {
            out(" ** SYNC DONE ** ");
            syncDone = true;
        }
    }
}
```

O método onBlock() é acionado em qualquer novo bloco recebido (seja antigo ou atual). EthereumJ representa e manipula blocos usando a classe org.ethereum.core.Block.

O método onSyncDone() é acionado assim que a sincronização é concluída, atualizando nossos dados locais do Ethereum.

# 5. Trabalhando com o Blockchain
Agora que podemos nos conectar à rede Ethereum e trabalhar diretamente com o blockchain, vamos mergulhar em várias operações básicas, mas muito importantes, que usaremos com frequência.

### 5.1. Enviando uma transação
Agora que nos conectamos ao blockchain, podemos enviar uma transação. Enviar uma transação é relativamente fácil, mas criar uma transação real é um tópico longo por si só:

```
ethereum.submitTransaction(new Transaction(new byte[]));
```

5,2 Acesse o objeto Blockchain
O método getBlockchain() retorna um objeto de fachada Blockchain com getters para buscar dificuldades de rede atuais e blocos específicos.

Como configuramos nosso EthereumListener na seção 4.3, podemos acessar o blockchain usando o método acima:

```
ethereum.getBlockchain();
```

### 5.3. Retorno de um endereço de conta Ethereum
Também podemos devolver um endereço Ethereum.

Para obter uma conta Ethereum - primeiro precisamos autenticar um par de chaves públicas e privadas no blockchain.

Vamos criar uma nova chave com um novo par de chaves aleatórias:

```
org.ethereum.crypto.ECKey key = new ECKey();
```

E vamos criar uma chave a partir de uma determinada chave privada:

```
org.ethereum.crypto.ECKey key = ECKey.fromPivate(privKey);
```

Podemos então usar nossa chave para inicializar uma conta. Ao chamar .init(), definimos uma ECKey e o Endereço associado no objeto Conta:

```
org.ethereum.core.Account account = new Account();
account.init(key);
```

# 6. Outras funcionalidades
Existem duas outras funcionalidades principais fornecidas pelo framework que não cobriremos aqui, mas que vale a pena mencionar.

Primeiro, temos a capacidade de compilar e executar contratos inteligentes Solidity. No entanto, criar contratos no Solidity e, posteriormente, compilá-los e executá-los é um tópico extenso por si só.

Em segundo lugar, embora a estrutura suporte mineração limitada usando uma CPU, usar um minerador de GPU é a abordagem recomendada, dada a falta de lucratividade do primeiro.

Tópicos mais avançados sobre o próprio Ethereum podem ser encontrados nos documentos oficiais.

# 7. Conclusão
Neste tutorial rápido, mostramos como se conectar à rede Ethereum e vários métodos importantes para trabalhar com o blockchain.