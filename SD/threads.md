# Threads
Dividem o trabalho pelos varios cores do pc. Assim, aumentamos a velocidade de 1 programa.

# Corridas
Quando 2 cores leem e escrevem na mesma variavel. Pode acontecer:
Core 1: read1 -> write2 -> read2 -> write3 -> ...
Core 2: read1 ->   ...  ->   ... ->  ...   -> write2
Logo, os valores finais não serao corretos

# Exclucao Mutua
Determinar **seccoes criticas** que sao partes do codigo em que apenas pode executar um thread de cada vez (tornar operacao atomica). Para isso usamos locks.
~~~
public interface Lock{
    public void lock(); //Fecha o lock
    public void unlock(); //Abre o lock
}
~~~
Na pratica, a estrutura do codigo é a seguinte:
~~~
public class X{
    private ReentrantLock lock=new ReentrantLock();

    public X metodo(){
        try{
            lock.lock();
            //codigo perigoso
        }finally{
            lock.unlock();
        }
    }
}
~~~
Nota: **ThreadID.get()** da-nos o identificador da thread.

# Propriedades da Computacao Assincrona
**Propriedades de protecao**: Ao pedir estas propriedades, estamos a dizer ao sistema que ha algo mau que nao deve acontecer. Ex. Exclusao mutua.
**Propriedades de animacao**: Indicam que um sistema está correta quando algo de bom acontece.
Temos de conseguir equilibrar estes 2 tipos de propriedades. Ex. Nao haver deadlock.

# Algoritmo de Peterson (2 threads)
Se tivesse um bool a dizer que a seccao critica estava livre mas duas threads a lessem ao mesmo tempo, isto levaria a um erro. 
Este algoritmo tem 1 array de flags que é acionado quando se quer usar a secaco critica. Se duas threads levantarem a flag ao mesmo tempo, nesse caso, temos de definir que uma das threads tem prioridade, tendo so essa acesso á seccao critica(impede que os 2 desistam em simultaneo, evitando deadlock). Se duas threads chegarem ao mesmo tempo, o ultimo a chegar é o que cede(variavel vitima=1). 

# Algoritmo de Filtro (Generalizar para N threads)
Temos de deixar apenas 1 entrar na seccao critica, atravez de um filtro de n-1 camadas.

| Thread 0 | Thread 1 | Thread 2 | Thread 3 | Vitima |
|----------------------------------------------------|
| 1 | 1 | 1 | 1 | 0
|   | 1 | 1 | 1 | 2
|   | 1 |   | 1 | 3
|   | 1 |   |   | 

Podemos substituir esta representacao por um array que diz quantas linhas da coluna estao ocupadas:
1->4->2->3

No final so temos uma thread que sobrevive, esta entra na zona critica. Pode haver ultrapassagens. Para tentar resolver esse problema, usamos o proximo algoritmo.

# Algoritmo da Padaria
A cada pedido é atibuido um numero de ordem. Se o numero de ordem for igual(pedido em simultaneo), desempatamos pelo numero da thread. O numero nunca é reposto a 0, é sempre incrementando.

# Memory order
Existem 2 Threads, uma que so escreve i=1 e depois j=1 e uma que le j e depois i. Os resultados possiveis da leitura sao 00,11,01 e 10. Como explicar o ultimo caso?
1. i=1 copiado para a cache
2. j=1 copiado para a cache
3. j copiado para memoria principal
4. ocorre a leitura por parte do segundo thread

Isto leva a que os algoritmos anteriores tenham problemas de exclusao mutua. Para que funcionem, temos de obrigar a escrever o que tudo o que está na cache, usando **barreiras de memoria**.
~~~
volatile int j;
//ou
AtomicInteger j;
~~~
Nota: O volatile aplicado a um array, atinge o array e nao os elementos nele contido. Para o fazer, usar o Atomic.

Nota: Nao preciasamos de usar o volatile nem o atomic nos exercicios que vamos realizar pois o reentranteLock faz com que a memoria seja sincronizada. Alem disso, uma variavel desse tipo quando é mexida faz com que se perda muito tempo.

# Synchronized
Quando declaramos o metodo como Synchronized, ele usa na pratica locks so que estao escondidos. Sao sintaticamente equivalentes. As unicas diferenças é que esta alternativa faz com que os unlocks tenham de ser pela ordem inversa dos locks, o que nem sempre é desejado, e que na entrada os threads podem entrar por qualquer ordem(podem se ultrapassar). O reentrantLock pode ser configurado para que essa ultrapassagem nao aconteça.

# Quando/Como usar locks(sincronizaçao)
Devemos usar normalmente locks nos gets, pois caso contrario poderiamos ler valores inconsistente.

# Paralelismo
Consideremos que a secao critica é pedir o numero que vamos testar se é primo. Vamos ver a diferença entre pedir 1 numero so ou 2 ao mesmo tempo.

1. Pedir numero -> Teste -> Pedir numero -> Teste
2. Pedir 2 numeros -> Teste -> Incrementar -> Teste

1. 
--++++--++++
  --++++--++++
    --++++--++++
Neste caso nao vale a pena pedir mais threads pois 1 deles nao estará a fazer trabalho util.

2. 
--++++++++++
  --++++++++++
    --++++++++++
      --++++++++++
        --++++++++++
Neste caso conseguimos ter mis threads em simultaneo. Como passam mais tempo fora da secao critica, esta pode ser usada por mais threads.
      
Devemos fazer com que o codigo das secçoes criticas seja pequeno em relacao ao codigo total e temos de fazer com que essa secção seja usada o minimo de vezes possiveis. **Devemos minimizar o numero de threads que querem aceder a uma secção critica bem como minimizar o tempo destas zonas.**

# Deadlock
Duas threads estao para sempre à espera um do outro. Para nao termos esse problemas temos de ordenar a aquisiçao de locks(**lock ordering**), independentemente das operaçoes que queiramos fazer.
EX: Se A quiser matar B e B quiser matar A, ambos tem de obter primeiro o lock do A e so depois o do B.

# Multiplos locks
----lock A----       -----lock B----
Se tivermos um observador no meio, consegue ver um estado incoerente.
----lock A---- 
  ----lock B----
Assim já resolveria o problema. Porem tornar isto mais eficiente
----lock A----
            ----lock B----
Isto é um caso concreto do 2PL

# 2PL (Two Phase locking)
1-> Todos os locks tem de ser feitos antes de qualquer unlock.
2-> Acesso a cada item de dados apensas é feito no periodo de tempo em que o lock é adquirido.

1 fase é crescimento, ou seja, adquire locks. A degunda fase é de decrescimo, ou seja, fazer unlocks.
O programa gerado é semelhante a 1 que tenha todos os locks adquiridos no incio e liberte todos no final.

Exp:
-----------lock A--------------
        --lock B--


# Locks vs Variavies
Varias threads que acedam ao mesmo item de dados concorrentemente devem adquirir o mesmo lock. Para o fazer:
-> O lock é uma variavel de instancia do objeto

Quando temos variaveis globais:
private **static** Lock l = new ReentrantLock();

# Readers-Writers locks
Para nao termos 1 lock por objeto.
~~~
interface ReadWriteLock{
  Lock readLock();
  Lock writeLock();
}
~~~
Varios leitores podem estar na seccao critica em simultaneo. No entante, 1 unico escritor pode excluir todos os outros escritores e os leitores.

# LockManager
Para nao termos muitos locks em simultaneo em memoria, temos um dispensor de locks.
~~~
interface LockManager{
  Lock lock(Object name);
  Lock unlock(Object name);
}
~~~
Se nã existir lock para o objeto que queremos, este cria-o. Aos air da seccao critica fazemos unlock e se ninguem o estiver a usar, podemos elimina-lo do mapa.

# Variaveis de condição
Queremos que uma thread espere por outras. Exp: Esperar para ter X jogadores para jogar. Tambem nao queremos estar numa espera ativa.

Note-se que temos de libertaar o lock enquanto estamos a espera para que outros possam entrar nesta seccao. Quando for acordado temos de repor o lock

~~~
Lock l = new ReentrantLock();
Condition c = l.newCondition();
~~~
**c.await;** -> Suspende o thread e liberta o lock associado
**c.signalAll();** -> Acorda todos os threads que foram suspendidos nessa condicao
**c.signal();** -> Acorda apenas uma das threads que foram suspendidas nessa condicao

Temos de ter em atencao que o lock seja utilizado pelos que acabaram de acordar e nao pelos novos que acabaram de chegar.
Para tentar resolver o problema, temos a seguinte estrutura:
~~~
void joinGame(){
  l.lock();
  ready++;
  while(ready<Min || playing>=Max){
    c.await();
  }
  playing++;
  c.signalAll();
  l.unlock();
}
~~~
O metodo c.await pode acordar de forma subtita pelo que devemos sempre usar o while para proteger o codigo.

# Ordering
Por omissao a thread que esta a espera pode ir para uma seccao critica livre. Se colocar-mos true no construtor do ReentrantLock, da mais prioridade os threads que estão à espera á mais tempo(no entanto nao garante que seja fifo).

As threads que adormecem perdem a vez na fila de adquiricao do lockn quando acordam, em comparacao com os que chegaram depois do numero max.

Solucao:
~~~
void joinTheGame(){
  l.lock();
  ticket=ready++;
  c.signalAll();
  while(ready<Min || playing>=Max || ticketTurn){
    c.await();
  }
  playing++;
  turn++;
  c.signalAll();
  l.unlock();
}
~~~
Nota: o sinalALL() está sempre correto se tivermos o while porem é menos eficiente.

# RW lock
Para o leitor, este tem de esperar enquanto houver um escritor a escrever. O escritor tem de esperar enquanto houver leitores a ler ou outro escritor a escrever.

~~~
void readLock(){
  l.lock();
  while(a_writer)
    c.await;
  nr_readers++;
  l.unlock();
}
void readUnlock(){
  l.lock();
  nr_readers--;
  if(nr_readers==0) //Otimizacao para so acordar os escritores se nao houver leitores
    c.signal();
  l.unlock();
}
void writeLock(){
  l.lock();
  while(a_writer || nr_readers>0)
    c.await();
  a_writer=true;
  l.unlock();
}
void writeUnlock(){
  l.lock();
  a_writer=false;
  c.signalAll();
  l.unlock();
}
~~~

# Soup counter -> Perceber quando usar signalAll ou signal
O consumidor tem de esperar pelo produtor se o balcao estiver vazio. Se o balao estiver cheio, o produtor nao pode colocar mais sopas nele.
Se o produtor colocar uma sopa no balcao ele necessita apenas de acordar um consumidor. Se o balcao estiver cheio e o consumidor tirar uma sopa, entao este pode apenas sinalizar um dos produtores para colocar uma sopa.
~~~
Object get(){
  l.lock();
  while(q.isEmpty())
    c.await();
  s=q.remove();
  c.signal();
  l.unlock();
}
void put(Object s){
  l.lock();
  while(q.size()>=Max)
    c.await()
  q.add(s);
  c.signal();
  l.unlock();
}
~~~
O problema é que ao usar o signal() desta forma podemos acordar tanto um produtor como um consumidor(estao na mesma variavel de condicao). **Bounded buffer** -> Este problema surge quando usamos diferentes condicoes associadas á mesma variavel de condicao.
Passamos a ter duas condicoes:
~~~
Object get(){
  l.lock();
  while(q.isEmpty())
    notEmpty.await();
  q.remove();
  notFull.signal();
  l.unlock();
}
void put(Object s){
  l.lock();
  while(q.size()>=Max)
    notFull.await()
  q.add(s);
  notEmpty.signal();
  l.unlock();
}
~~~

# Continuar RW lock
Se entrarem muitos leitores os escritores que aparecem podem nunca conseguir entrar. Uma alternativa seria dar prioridade aos escritores. Porem, assim, podemos fazer o inverso, ou seja, os leitores podem bloquear os leitores. 

Solucao: Proibe que hajam novos escritores a entrar e espera que todos os leitores que estavam a espera acabem de ler. Competem de uma forma mais justa.
~~~
void readLock(){
  l.lock();
  while(a_writer)
    cw.await;
  nr_readers++;
  l.unlock();
}
void readUnlock(){
  l.lock();
  nr_readers--;
  if(nr_readers==0) //Otimizacao para so acordar os escritores se nao houver leitores
    cr.signal();
  l.unlock();
}
void writeLock(){
  l.lock();
  while(a_writer)
    cw.await();
  a_writer=true;
  while(nr_readers>0)
    cr.await();
  l.unlock();
}
void writeUnlock(){
  l.lock();
  a_writer=false;
  cw.signalAll();
  l.unlock();
}
~~~
