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







