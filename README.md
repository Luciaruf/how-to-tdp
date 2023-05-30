# 📝 TODO: TDP

## 🎒 Roba da portare / Todo prima dell'esame

- Pubblica su GitHub tutti i progetti che vuoi controllare durante l'esame e pinnali nella homepage
- :exclamation: USB con:
  - Materiale TDP e appunti (progetti, laboratori, slides, ***appunti! (time handling!!!)*** e tutto quello che vuoi)
  - File TXT con password Polito, password GitHub, e chiave SSH GitHub
- :exclamation: Cellulare con password Polito e password GitHub + Autenticatore a 2 fattori etc.
- :exclamation: Carta e penna
- Auricolari e Acqua
- Fai un backup dell'USB su WhatsAppWeb per recuperare i dati in caso di necessità
- Prima dell'esame, apri una pagina di Google Chrome in modalità *incognito* e apri
  - [WhatsApp Web](https://web.whatsapp.com/)
  - [Phind](https://www.phind.com/)
  - [Pagina Web Ufficiale di TdP](https://elite.polito.it/teaching/03fyz-tdp/intro)
  - [GitHub](https://github.com/nMaax)
  - GitHub Classroom

## 📝 Da controllare alla fine

- Assicurati di aver definito `hashCode` e `equals` in ogni java bean
- Pulisci il grafo ogni volta che fai clic sul pulsante "Crea grafo"
- Pulisci dati dentro il Model e text field nel Controller se devono essere puliti durante l'esecuzione
- Controlla possibili errori di input nel Controller e trova modi per rompere il tuo codice (vedi a [fine punto 1]())
- Controlla attentamente le tabelle e la struttura del db utilizzata
- Controlla attentamente di utilizzare il tipo di dato giusto durante il recupero dei dati dal dao (double, int, String, LocalDateTime, ...)
- Fai dichiarazioni di Sysout pulite per debuggare sul momento, ma non spenderci troppo tempo
- Usa `try (Connection conn = ...)` o chiudi `conn` alla fine di ogni blocco `try`
  
```java
/*
 *  try(...) si chiama try-with-resources, 
 *  è un tipo di comando try aggiunto in java 7 che chiude 
 *  automaticamente l'oggetto dichiarato al suo interno una volta che il
 *  blocco try è terminato (utilizzando il suo relativo metodo .close()),
 *  è una soluzione più solida e sicura di chiamare il comando .close() manualmente
 *  in quanto, anche se viene lanciato un errore o eccezione, tale oggetto verrà chiuso lo stesso
 *  Nota: try-w-resources funziona solo se la classe dell'ogetto che viene richiamato dentro le parentesi 
 *  implementa l'interfaccia AutoCloasble: Connection infatti la implementa come è visibile nelle seguenti risorse
 *  Doc try-w-resources: https://docs.oracle.com/javase/tutorial/essential/exceptions/tryResourceClose.html
 *  Doc Connection implementa AutoClosable: https://docs.oracle.com/javase/8/docs/api/java/sql/Connection.html
 *  StackOverflow che spiega il funzionamento di try-w-resources: https://stackoverflow.com/questions/10115282/new-strange-java-try-syntax#answers
 */
```

- Eventualmente dichiara la complessità di ogni algoritmo che pensi meriti di essere citato (solo per mostrare che sai che questi algoritmi potrebbero non essere ottimali)
  - Se hai tempo forse puoi fare un controllo tic-toc da qualche parte
- Dopo ogni esame/simulazione elimina le chiavi github che potrebbero essere memorizzate in eclipse o altrove!

***Note***
> :zap: La seconda parte dell'esame non sarà valutata (solo) sulla base dei semplici output che il tuo algoritmo fa ma anche sulla logica che hai implementato, se per esempio il tuo algoritmo ricorsivo "esplode" facilmente non farti prendere dal panico, sarà lecito lo stesso e verrà valutato solo se ci sono delle vere e proprie fallacie logiche!

## 📖 Prima Parte: DAO, JDBC, jGraphs (+ JavaFX, MVC, ConnectionPooling, Best Practices, ...)

### 1. Skeleton of model

```java
Graph<?,?> graph, 
List<Node> allNodes, 
Dao dao
Map<Integer, Node> idMap // Only if necessary!

public Model() { 
    graph = new Graph<>(Edge.class); 
    allNodes = new ArrayList<>; 
    dao = new Dao(); 
    idMap = new HashMap<>(); 
    ...; // lazy
}

private void loadNodes() -> {
    // Se o la mappa, o la lista, sono vuote (anche solo una delle due) allora 
    // significa che si è verificato un errore nel caricamento di tali dati dal DB
    // dunque è meglio ricaricare tutto da capo
    if (this.allNodes.isEmpty() || this.idMap.isEmpty()){ 
        this.allNodes = this.dao.getAllNodes;
        for (Node n : this.allNodes) this.idMap.put(n.getId(), n); 
    }
    System.out.println(...) // "> Caricati tutti i NODI" + allNodes.size() + " | " + nodeIdMap.keySet().size()
}

public void buildGraph() -> {

    // Clean the graph if necessary
    if (graph.vertexSet().size() > 0) graph = new SimpleGraph<>(DefaultEdge.class);

    loadNodes();
    Graphs.addAllVertices(Graph, allNodes);
    System.out.println(...) // "> Caricati i NODI nei vertici del grafo: " + graph.vertexSet().size()
    //TODO load Edges
}
```

### 2 Make Queries

- Watch out for Double Couples and Self Couples! --> If the graph is oriented, weighted or do not alow self-loop this could broke your code!
- Do not make super-hard queries, in case it happens ask teachers!
- Make sure to work on the right tables and data
- Check that both on java and on sql you get the right cardinality of the collection (Collection.size() and the number of rows in HeidiSQL must coincide)

Watch out for *self-loops* and *double-pairs*!

```sql
    SELECT e1.object_id AS o1, e2.object_id AS o2, COUNT(*) AS weight
    FROM exhibition_objects e1, exhibition_objects e2
    WHERE e1.exhibition_id = e2.exhibition_id
    AND e1.object_id > e2.object_id -- Double pair (and thus, self-loops) avoided :)
    GROUP BY o1, o2
```

```sql
    SELECT s1.playerID, s2.playerID, s1.teamID, s1.teamCode
    FROM salaries s1, salaries s2
    WHERE s1.teamID = s2.teamID
    AND s1.playerID <> s2.playerID -- Self-loop avoided :)
```

***Note***
> :zap: Double pair are automaticaly avoided if you use an undirected graph, but if possible avoid them directly in your query/dao to make a cleaner code!

### 2.5 Create unimplementhed DAO methods and Edge Auxiliary Class

```java

/*
 *  try(...){} si chiama "try-with-resources", 
 *  è un tipo di comando try aggiunto in java 7 che chiude 
 *  automaticamente l'oggetto dichiarato al suo interno una volta che il
 *  blocco try è terminato (utilizzando il suo relativo metodo .close()),
 *  è una soluzione più solida e sicura di chiamare il comando .close() manualmente
 *  in quanto, anche se viene lanciato un errore o eccezione, tale oggetto verrà chiuso lo stesso
 * 
 *  Nota: try-w-resources funziona solo se la classe dell'ogetto che viene richiamato dentro le parentesi 
 *  implementa l'interfaccia AutoCloasble: Connection infatti la implementa come è visibile nella documentazione ufficiale di java disponibile qui sotto
 * 
 *  try-w-resources: https://docs.oracle.com/javase/tutorial/essential/exceptions/tryResourceClose.html
 *  Connection implementa AutoClosable: https://docs.oracle.com/javase/8/docs/api/java/sql/Connection.html
 *  Utente StackOverflow che spiega il funzionamento di try-w-resources: https://stackoverflow.com/questions/10115282/new-strange-java-try-syntax#answers
 * 
 * try-with-resources NON equivale a 
   
    try {
        // Stuff...
    } finally {
        conn.close();
    }
 
 * try-with-resources è più sicuro!
 * come specificato nella stessa documentazione di java, 
 * disponibile qui: https://docs.oracle.com/javase/tutorial/essential/exceptions/tryResourceClose.html#:~:text=However%2C%20this%20example,your%20program%27s%20resources.
 */

void Collection<Stuff> loadStuff(Map<Integer, Node> idMap, ...) {

    String sql = "SELECT s.playerID "
                + "FROM salaries s "
                + "WHERE s.`year` = ? "
                + "AND s.salary > ?";
    
    // N.B. conn viene chiuso automaticamente in un try-with-resources in quanto implementa l'interfaccia AutoClosable,
    // si veda il commento multi-linea sopra per una spiegazione più dettagliata di try-with-resources
    // https://docs.oracle.com/javase/tutorial/essential/exceptions/tryResourceClose.html
    try (Connection conn = DBConnect.getConnection();) {

        PreparedStatement st = conn.prepareStatement(sql);
        st.setInt(1, ...);
        ResultSet rs = st.executeQuery();

        List<People> result = new ArrayList<People>();
        while (rs.next()) {
            Node n = idMap.get(rs.getString("playerID"));
            result.add(n);
        }

        return result;

    } catch (SQLException e) {
        e.printStackTrace();
        return null;
    }
}

```

```java

void Collection<NodePair> loadEdges(Map<Integer, Node> idMap, ...) {
    
    String sql = "SELECT g1.playerID g1, g2.playerID g2 "
            + "FROM appearances g1, appearances g2 "
            + "WHERE g1.teamID = g2.teamID "
            + "AND g1.`year` = g2.`year` "
            + "AND g1.`year` = ? "
            + "AND g1.playerID <> g2.playerID"; // Watch out for Self-Loops and Double-Pairs!

    // N.B. conn viene chiuso automaticamente in un try-with-resources in quanto implementa l'interfaccia AutoClosable,
    // si veda il commento multi-linea sopra per una spiegazione più dettagliata di try-with-resources
    // https://docs.oracle.com/javase/tutorial/essential/exceptions/tryResourceClose.html
    try (Connection conn = DBConnect.getConnection();) {

        PreparedStatement st = conn.prepareStatement(sql);
        st.setInt(1, ...);
        ResultSet rs = st.executeQuery();

        List<NodePair> result = new ArrayList<>();
        while(rs.next()) {
            Node n1 = idMap.get(rs.getInt("g1"));
            Node n2 = idMap.get(rs.getInt("g2"));;
            NodePair np = new NodePair(n1, n2);
            result.add(np);
        }
        return result;

    } catch (SQLException e) {
        e.printStackTrace();
        return null;
    }
}
```

#### Edge auxiliary class

| :exclamation:  Remember to define `hashCode` and `equals` (and eventually `compareTo`) in each new Java Bean you define, just in case |
|------------------------------------------------------------------------------------------------------|

If you want to make two pairs containing one (a, b) and the second (b, a) to look the same from .equals() and hashCode() perspective consider the following options:

1. Before saving the pair into the object, sort them in the constructor

    ```java

    public NodePair(Node a, Node b) {
        super();
        if (a.compareTo(B) > 0) { //or Integer.compareTo(a.getID(), b.getID())
            this.a = a;
            this.b = b;
        } else {
            this.a = b;
            this.b = a;
        }
    }

    ```

2. Use specifc equals() and hashCode() methods

    ```java

    @Override
    public int hashCode() {

        // https://stackoverflow.com/questions/1536393/good-hash-function-for-permutations

        // XOR [ a.hashCode() ^ b.hashCode() ] , 
        // or SUM [ a.hashCode() + b.hashCode() ] , 
        // or MULTIPLY [ a.hashCode() * b.hashCode() ]
        // or use abc's propose: hashNumbe = SUM[ R + 2*i ] / 2 
        // where R is a arbitrary odd number > 1 and i is a generic element of the set
        return (abc(a.hashCode()) + abc(b.hashCode()) / 2);
    }

    private double abc(int a) {
        int R = 1779033703;
        return R + 2*a;
    } 

    @Override
    public boolean equals(Object obj) {
        if (this == obj)
            return true;
        if (obj == null)
            return false;
        if (getClass() != obj.getClass())
            return false;
        NodePair other = (NodePair) obj;
        return ( Objects.equals(a, other.a) && Objects.equals(b, other.b) ) ||
            ( Objects.equals(a, other.b) && Objects.equals(b, other.a) );
    }


    ```

If instead you just need a simple NodePair class use the following standard java bean

```java
public class NodePair {

    Node a;
    Node b;

    public NodePair(People a, People b) {
        super();
        this.a = a;
        this.b = b;
    }

    public NodePair getA() {
        return a;
    }

    public NodePair getB() {
        return b;
    }

    @Override
    public int hashCode() {
        return Objects.hash(a, b);
    }

    @Override
    public boolean equals(Object obj) {
        if (this == obj)
            return true;
        if (obj == null)
            return false;
        if (getClass() != obj.getClass())
            return false;
        NodePair other = (NodePair) obj;
        return Objects.equals(a, other.a) && Objects.equals(b, other.b);
    }

    @Override
    public String toString() {
        return "NodePair [a=" + a + ", b=" + b + "]";
    }

}
```

### 2.5.5 Test DAO Methods (if you need idMaps to test these methods make idMaps Getters in the model!)

```java
Model m = new Model();
m.buildGraph();

DAO dao = new DAO();

List<NodePair> nodes = dao.loadEdges(m.getIdMap(););
System.out.println(nodes.size());
```

### 3. Update buildGraph() with new DAO methods to load edges

```java
void buildGraph() -> { 

    // TODO Preliminar stuff ...

    loadNodes();
    Graphs.addAllVertices(Graph, allNodes);
    System.out.println(...) // "> Caricati i nodi nei vertici del grafo: " + graph.vertexSet().size()

    // Load Edges
    for (NodePair p : dao.getNodePairs(..., idMap)) {
        Node n1 = p.getNode1();
        Node n2 = p.getNode2();
        //if (!graph.containsVertex(n1)) graph.addVertex(n1);
        //if (!graph.containsVertex(n2)) graph.addVertex(n2);
        graph.addEdge(n1, n2);
    }
    System.out.println(...) // "> Caricati gli archi del grafo: " + graph.edgeSet().size()

}
```

### 3.5 Test buildGraph

```java
Model m = new Model();
m.loadPeople();
m.buildGraph();
```

### 4. Implement logical operation on graphs (DepthFirstIterator, BreadthFirst, ConnectivityInspector, ...)

```java
// Sample DepthFirstIterator | BreadthFirst manual implementation

DepthFirstIterator<Node, DefaultEdge> iterator = 
                new DepthFirstIterator<>(graph, rootNode);

List<Node> result = new ArrayList<>();
while (iterator.hasNext())
    result.add(iterator.next());
return result;

```

```java

// Sample findPath algorithm using a DepthFirstIterator

public findPath(Node source, Node sink) {

    BreadthFirstIterator<Fermata, DefaultEdge> iterator = 
            new BreadthFirstIterator<>(graph, source);

    List<Node> reachableNodes = new ArrayList<>();
    while (iterator.hasNext()) {
        Node n = iterator.next();
        reachableNodes.add(n);
    }

    List<Fermata> path = new ArrayList<>();
    Node currentNode = sink;
    percorso.add(sink);
    DefaultEdge e = iterator.getSpanningTreeEdge(currentNode);

    while (e != null) {s
        Node befNode = Graphs.getOppositeVertex(graph, e, currentNode);	
        // Add at index 0 in order to don't have a list backwards
        path.add(0, befNode);
        currentNode = befNode;
        e = iterator.getSpanningTreeEdge(currentNode);
    }

}
```

```java
// Sample ConnectivityInspector implementation

ConnectivityInspector<Node, DefaultEdge> inspector = new ConnectivityInspector<>(graph);
return new ArrayList<>(inspector.connectedSetOf(rootNode));

```

### 4.5 Test logical operations on the graph

### 5. Implement these new methods on the controller (check bad inputs, errors and data-cleaning procedures on the controller!)

Here is a simple list of usual things that could break your code:

1. Inserting wrong type of data as input (int instead of a char or double instead of an int)
2. Inserting data with spaces before, after or in between itself ("   69", "420  ", ...) as input
3. Inserting nothing ("") as input
4. Inserting data below a lower bound (e.g. negative numbers, lookout for limits in data range in the DB!)
5. Inserting data above an upper bound (e.g. really large numbers, lookout for limits in data range in the DB!)
6. Clicking buttons two or more times when each time data is not reseted (e.g. click two times "Make Graph" will load double vertex, edges, ...)
7. Clicking some buttons when other have not already been clicked (e.g. click "Find shortest path" when the user didnt already click "Make graph") - Hint: Use ```javafx.obj.setDisable(false | true)```

### 6. Eventually make getters for each attribute of the model class made so far

Model -> `getAllPeople`, `getIdMap`, `getGraph`, ...

Make model unmutable and do not make getter for stuff you dont want to show, use private keywords for certain methods as well

## 🐍 Seconda Parte: Algoritmo di Ricorsione o Simulatore ad Eventi Discreti

### 7. Combinatory Recursion Algorithm

***Note***
> :bulb: If the recursive algorithm consist of a MIN-MAX function then know that you could avoid recursion and use Operative Research instead! But you would need a java library that imlements Simplex Method etc. so not really doable during an exam

```java
// Schema from teachers

/*
Consider using HashSet in your code. The use of `HashSet` in your code depends on the requirements of your algorithm and the characteristics of the data you're working with. Here are a few points to consider:

    1. **Duplicate checking:** If you need to check for duplicates frequently, a `HashSet` could provide a performance improvement because it offers O(1) complexity for the `contains` operation, while an `ArrayList` offers O(n) complexity. However, in your current pseudocode, I don't see a need for such operation.

    2. **Order of elements:** If the order of the elements is important for your algorithm (for instance, in the `best` list), then you should stick with `ArrayList` or other types of `List`, as `HashSet` doesn't maintain the order of elements. If order is not important and you need to eliminate duplicates, a `HashSet` can be a good choice.

    3. **Filtering operation:** The `filter` function could potentially benefit from a `HashSet`. If you're checking whether an element exists in the `partial` list, a `HashSet` would offer better performance.

    4. **Memory usage:** `HashSet` takes more memory than `ArrayList` because it needs to maintain a separate data structure (a hash table) to provide constant-time performance.

    In conclusion, you need to consider these factors based on the specific characteristics of your problem and data. The use of `HashSet` can be an improvement in some cases, but it could also be a disadvantage depending on your needs. Always consider the trade-offs before deciding.
*/

/*
Rispondere alle seguenti domande:
    - Cosa rappresenta il "livello" nel mio algoritmo ricorsivo?
    - Com'è fatta una soluzione parziale?
    - Come faccio a riconoscere se una soluzione parziale è anche completa?
    - Data una soluzione parziale, come faccio a sapere se è valida o se non è valida? (nb. magari non posso)
    - Data una soluzione completa, come faccio a sapere se è valida o se non è valida?
    - Qual è la regola per generare tutte le soluzioni del livello+1 a partire da una soluzione parziale del livello corrente?
    - Qual è la struttura dati per memorizzare una soluzione (parziale o completa)?
    - Qual è la struttura dati per memorizzare lo stato della ricerca (della ricorsione)?
*/

// New Attributes

private List<Node> best; // The current best solution
private double bestNumber; // A number that represents the goodness of the solution, faster that using data structures with methods
private int limit; // Level limit

// Setup

public List<Node> start(int limit) {
    this.best = new ArrayList<>();
    this.bestNumber = 0;
    this.limit = limit; 
    List<Node> partial = new ArrayList<>();
    recursive(partial);
    return this.best;
}

// Recursive algorithm

public void recursive(List<Node> partial) {
    
    // Comandi da fare sempre, molto raro
    doAlways();

    // Condizione di terminazione
    if (something > limit) return;
    
    // Controllo della migliore soluzione
    double currentNumber = calcNumber(partial);
    if (isComplete(current) && currentNumber > bestNumber) {
        best.clear();
        best.addAll(partial);
        bestNumber = currentNumber;
    }
    
    clean(nodes) // Pulisci i nodi su cui ciclare prima di iniziare il for, se possibile

    // Ricorsione 
    for (Node n : nodes) {
        // Filtro in entrata
        if (filter(partial, n)) {
            partial.add(p);
            // Maybe instantiating a new ArrayList<>(partial) could be a good option?
            // Note: Instantiating a new list at each step can be memory intensive if your recursion depth is large. 
            // If partial doesn't need to be immutable across recursive calls, it would be more efficient to
            // avoid new instantiations and reuse the list as you're currently doing.
            recursive(partial); 
            // Backtracking
            partial.remove(partial.size()-1); // Keep in mind that this will only work if partial has not been shuffled before!
        }
    }
}

// Filters and controls

public void doAlways() {
    //TODO
}

public boolean isComplete(List<Node> partial) {
    return true;
}

public void clean(List<Node> nodes) {
    //TODO
}

public double calcNumber(List<Node> partial) {
    //TODO
    return 0.0; //placeholder return
}

public boolean filter(List<Node> partial, Node n) {
    //TODO
    return false; //placeholder return
}

```

### 8. Discrete Event Simulator

```java
// Personal Schema

```
