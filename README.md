# Graph Mining - Game of Thrones (GoT)

## Intro

* This project was developed as a form of evaluation of the course "Graph Mining" in the MBA in Data Science at the University of Fortaleza.

## Objective

* The goal of this project is to identify the main character of the TV show GoT from the episode scripts using network analysis 

## Dataset

* The dataset to be used is in .txt format and is composed of the script of the episodes of all seasons:

    * 8 seasons with a total of 73 episodes

![keys](/images/keys.png)

![values](/images/values.png)

* Two .txt files will also be used as a complementary data set, which contain the names of the cast and extra characters, including alias or misspelled words, grouped with their respective character.

* Cast characters example:

![cast](/images/cast.png)

* Extra characters example:

![extra](/images/extra.png)

## Data Engineering

* The dataset cleaning was performed using the following strategies:

    * Join the text of all episodes into one list and exclude the \n

    * Transform the extra character's data from a list with one string (with all the words) to a list of elements (or strings), in which each element will be a word. As well as removing the " and \n

    ![extra_clean](/images/extra_clean.png)

    * Transform the cast character's data into a nested list, in which each list will be composed of the character's aliases and misspellings. As well as removing the \n

    ![cast_clean](/images/cast_clean.png)

## Nodes and Edges selection

### Nodes

* The strategy to define the nodes consisted in checking the names present before the script referring to the character's speech.

* The example below demonstrates the speech of a character, which is preceded by its respective name.

        'WAYMAR ROYCE: What d’you expect? They’re savages. One lot steals a goat from another lot and before you know it, they’re ripping each other to pieces.\n']

* The goal is to extract only the name ```WAYMAR ROYCE```

* The loop created to extract the nodes, consists of:
    * Searches for the pattern in the text

    * Adds the node to the list when it finds the pattern and if it's not a extra character

    * Look for the node in the cast character list:

        * If node exists in the cast character list, it considers only one name from the aliases

        * If not, the node is added to the nodes list

* Frequency of each node in the text in descending order:

        [('tyriom', 1850),
        ('john', 1250),
        ('daenarys stormborn', 1085),
        ('cersei', 1035),
        ('jaime', 978),
        ('cut to', 905),
        ('little bird', 842),
        ('arya stark', 803),
        ('dav os', 521),
        ('theon', 491)]

* Through an initial analysis, it can be observed that Tyriom is the most mentioned name in the scripts, followed by John and Daenarys. Possibly, these are the main characters of the story

* Finally, a list was created with the unique nodes, totaling **322** nodes

### Context Identification

* Before defining the connections (or edges) between nodes it is necessary to identify the context of each line of text, and for this a function was created.

* The function looks for the same pattern used to identify the nodes, that is, if the line has a text and : at the beginning, limited to 20 characters so as not to bring up phrases, as shown below:

        'WAYMAR ROYCE: What d’you expect? They’re savages. One lot steals a goat from another lot and before you know it, they’re ripping each other to pieces.\n']

* When the pattern is found, the line is defined as a dialog and some conditions are tested that concern the description of the scene and not a character's speech.

        CUT TO: GREAT WEIRWOOD TREE - BACK ENTRANCE

        Tell me something: Do you still believe good soldiers make good kings?

        EXT: THE STREETS OF KING'S LANDING

        INT: THE RED KEEP CELLAR

* If any of these conditions are true, the line is not considered a dialog.

        def IsDialog(text):
            pattern = r'^.{1,20}(?=\:)'
            dialog = re.match(pattern, text)

            if dialog:
                patt_cut = r'^cut to(?=\:)'
                cut_to = re.match(patt_cut, text, flags = re.I)

                patt_something = r'^tell me something(?=\:)'
                tell_me_something = re.match(patt_something, text, flags = re.I)
                
                patt_ext = r'^ext(?=\:)'
                exterior = re.match(patt_ext, text, flags = re.I)
                
                patt_int = r'^int(?=\:)'
                interior = re.match(patt_int, text, flags = re.I)

                if cut_to or tell_me_something or exterior or interior:
                    return False
                
                else:
                    return True
            else:
                return False

* In short, this function aims to filter the text with only those lines that are character lines.

### Edges

* To define the links between nodes a loop was created that returns a dictionary:

    * key: tuple with the names of 2 characters (link or edge)

    * value: absolute frequency of the tuple

* The loop logic for defining what is a connection between two characters consists of:

    * The ```IsDialog``` function is applied to the text to identify lines that are character speech;

    * The link is created between the line being iterated (character's speech in line X) with previous line (character's speech in line X-1)

    * In this way, a tuple is created with the names of the two characters and their frequency is counted

* For example: in the text below, the loop would create two tuples:

**Text:**        

        JOFFREY: Rhaenyra Targaryen was murdered by her brother, or rather his dragon. It ate her while her son watched. What's left of her is buried in the crypts right down there.

        CERSEI: The ceremony is traditionally held in the main sanctum, which seats seven hundred comfortably.

        OLENNA: There appears to be a good deal of room elsewhere on the premises for everyone else.

**Result:**

        (JOFFREY, CERSEI) (CERSEI, OLENNA)

In short, the dictionary resulting from the loop will have the edges as keys (links) and the respective weight of the edge as value (absolute frequency)

![edges](/images/edges.png)

## Network Creation

* In this section the network is created using the NetworkX library, which is a Python package for the creation, manipulation, and study of the structure, dynamics, and functions of complex networks.

* The network was created from the edge dictionary, in which the key represents the edge between two nodes (tuple) and the value represents the weight of the edge (absolute frequency)

![network](/graphs/network.png)

* The nodes of the possible main characters were colored green. It can be seen that they are very central in the network and have several edges

        possible_main_characters = ['jon', 'tyrion', 'daenerys', 'jaime', 'cersei', 'sansa']

## Centrality Measures

* The centrality measures aim to investigate which would be the most important nodes (or characters) in a network.

* The main hypothesis is that the more relations and articulation of relations a node has, the greater its degree of importance and relevance to the structural pattern of the network.

* However, the number of relationships will not always be the main measure for network analysis. In other cases, for example, the main question is not the number of relations, but the position in which the node is located, or the relations of greatest relevance or weight for a given context.

* Thus, some measures of centrality are calculated in this section, namely:

    * Degree Centrality

    * Eigenvector Centrality

    * PageRank

    * Closeness Centrality

### Degree Centrality

* Degree centrality is defined as the number of links incident upon a node (i.e., the number of ties that a node has). If the network is directed (meaning that ties have direction), then two separate measures of degree centrality are defined, namely, indegree and outdegree.

* Indegree is a count of the number of ties directed to the node (head endpoints) and outdegree is the number of ties that the node directs to others (tail endpoints). In such cases, the degree is the sum of indegree and outdegree.

* Analyzing the degree of centrality, it is possible to observe that 4 nodes stand out from the others and present the largest number of connections, namely:

        [('jon', 42),
        ('tyrion', 41),
        ('daenerys', 41),
        ('jaime', 40)]

* Degree centrality is defined as the amount of connections a node has, and consequently, its interpretation is based on its relative importance to mobilize or trigger other nodes.

* According to the degree centrality, we can define a group of 4 main characters that stand out from the others by the amount of connections. These characters are the ones who have more interaction with other characters during the plot.

### Eigenvector Centrality

* Eigenvector Centrality is an algorithm that measures the transitive influence of nodes. Relationships originating from high-scoring nodes contribute more to the score of a node than connections from low-scoring nodes. A high eigenvector score means that a node is connected to many nodes who themselves have high scores.

![eigenvector](/graphs/eigenvector.png)

* The purpose of eigenvector centrality is to measure the importance of a vertex as a function of the importance of its neighbors. This means that even if a vertex is connected to only a few other vertices in the network (thus having a low degree centrality), these neighbors can be important and, consequently, the vertex will also be important, obtaining a high eigenvector centrality.

* Analyzing the ranked eigenvector score of the nodes and considering the top 4 results, we have:

        [('tyrion', '0.30'),
        ('jon', '0.29'),
        ('jaime', '0.27'),
        ('sansa', '0.24')]

* Comparing the result above with the result of degree centrality, we can observe that the node "sansa" appeared in place of "daenerys". Thus, it can be concluded that although the node "sansa" has less connections (or edges) than the node "daenerys", it has more relevant connections, that is, with higher scores.

### PageRank

* PageRank (PR) is an algorithm used by Google Search to rank web pages in their search engine results and is a variant of the eigenvector centrality measure.

* It was introduced by Larry Page and was first used to rank web pages in the Google search engine. Nowadays, it is more and more used in many different fields, for example in ranking users in social media, graphs, complex networks, etc.

* According to Google:

```PageRank works by counting the number and quality of links to a page to determine a rough estimate of how important the website is. The underlying assumption is that more important websites are likely to receive more links from other websites```

![pagerank](/graphs/pagerank.png)

* PageRank computes a ranking of the nodes in the graph G based on the structure of the incoming links. The ranking of the nodes calculated by PageRank showed the same top four results as the degree centrality, but in a different order.

        [('tyrion', '0.07'),
        ('jaime', '0.05'),
        ('daenerys', '0.05'),
        ('jon', '0.04')]

### Closeness Centrality

* Closeness centrality indicates how close a node is to all other nodes in the network. It is calculated as the average shortest distance from each vertex to each other vertex. Specifically, it is the inverse of the average shortest distance between the vertex and all other vertices in the network.

![closeness](/graphs/closeness.png)

* Proximity centrality measures how central a node is, i.e. the more central a node is, the closer it is to all other nodes.

* The most central nodes in the network are:

        [('jon', '0.47'),
        ('tyrion', '0.46'),
        ('jaime', '0.45'),
        ('sansa', '0.45')]

## Conclusion

* The objective of this project was to define which is the most important character in the plot from the episode scripts using network analysis. In this way, the centrality measures were calculated in order to indicate which would be this character.

* The first ranking node for each method came down to two characters, as follows:

    * Degree Centrality = Jon

    * Eigenvector Centrality = Tyrion

    * PageRank = Tyrion

    * Closeness Centrality = Jon

* Then, from the methods employed, it can be conclude that there is more than one main character, namely: Jon and Tyrion. However, by extending the ranks of the rankings it is possible to observe the following names frequently:

    * Jaime
    
    * Sansa
    
    * Daenerys

* So, if we consider a group of main characters instead of just one, we can come to the conclusion of the following group:

    * Jon
    
    * Tyrion
    
    * Jaime
    
    * Sansa
    
    * Daenerys
![themes_avg1](/graphs/themes_avg1.png)
