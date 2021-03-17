## Lab 6 - Scientific Computation 3/12/2019

Once you have installed the tools and familiarized yourself with the networkx code, you are ready to begin recording your lab in your lab notebook. We will start with the problem of words of length 5 and words of length 4.

1. The word game implementation you downloaded already takes words of length 5.

<!-- **_Results_** -->
        Loaded words_dat.txt containing 5757 five-letter English words.
        Two words are connected if they differ in one letter.
        Graph has 5757 nodes with 14135 edges
        853 connected components
        Shortest path between chaos and order is
        chaos
        choos
        shoos
        shoes
        shoed
        shred
        sired
        sided
        aided
        added
        adder
        odder
        order
        Shortest path between nodes and graph is
        nodes
        lodes
        lores
        lords
        loads
        goads
        grads
        grade
        grape
        graph
        Shortest path between moron and smart is
        moron
        boron
        baron
        caron
        capon
        capos
        capes
        canes
        banes
        bands
        bends
        beads
        bears
        sears
        stars
        start
        smart
        Shortest path between flies and swims is
        flies
        flips
        slips
        slims
        swims
        Shortest path between mango and peach is
        mango
        mange
        marge
        merge
        merse
        terse
        tease
        pease
        peace
        peach
        Shortest path between pound and marks is
        None


2. Now generate a new version that takes words of length 4 and test with: 4. `cold` to `warm` 5. `love` to `hate` 5. `good` to `evil` 5. `pear` to `beef` 5. `make` to `take`

        Loaded words_dat.txt containing 5757 five-letter English words.
        Two words are connected if they differ in one letter.
        Graph has 2174 nodes with 8040 edges
        129 connected components
        Shortest path between cold and warm is
        cold
        wold
        word
        ward
        warm
        Shortest path between love and hate is
        love
        hove
        have
        hate
        Shortest path between good and evil is
        None
        Shortest path between pear and beef is
        pear
        bear
        beer
        beef
        Shortest path between make and take is
        make
        take

        import gzip
        from string import ascii_lowercase as lowercase

        import networkx as nx

        #-------------------------------------------------------------------
        #   The Words/Ladder graph of Section 1.1
        #-------------------------------------------------------------------


        def generate_graph(words):
            G = nx.Graph(name="words")
            lookup = dict((c, lowercase.index(c)) for c in lowercase)

            def edit_distance_one(word):
                for i in range(len(word)):
                    left, c, right = word[0:i], word[i], word[i + 1:]
                    j = lookup[c]  # lowercase.index(c)
                    for cc in lowercase[j + 1:]:
                        yield left + cc + right
            candgen = ((word, cand) for word in sorted(words)
                    for cand in edit_distance_one(word) if cand in words)
            G.add_nodes_from(words)
            for word, cand in candgen:
                G.add_edge(word, cand)
            return G


        def words_graph():
            """Return the words example graph from the Stanford GraphBase"""
            fh = gzip.open('words4_dat.txt.gz', 'r')
            words = set()
            for line in fh.readlines():
                line = line.decode()
                if line.startswith('*'):
                    continue
                w = str(line[0:4])
                words.add(w)
            return generate_graph(words)


        if __name__ == '__main__':
            G = words_graph()
            print("Loaded words_dat.txt containing 5757 five-letter English words.")
            print("Two words are connected if they differ in one letter.")
            print("Graph has %d nodes with %d edges"
                % (nx.number_of_nodes(G), nx.number_of_edges(G)))
            print("%d connected components" % nx.number_connected_components(G))

            for (source, target) in [('cold', 'warm'),
                                    ('love', 'hate'),
                                    ('good', 'evil'),
                                    ('pear', 'beef'),
                                    ('make', 'take')]:
                print("Shortest path between %s and %s is" % (source, target))
                try:
                    sp = nx.shortest_path(G, source, target)
                    for n in sp:
                        print(n)
                except nx.NetworkXNoPath:
                    print("None")

3. Implement a variation where we consider two words (nodes) to be adjacent if there is a one letter difference without regard to ordering. You will need to change the edit*distance_one function to disregard letter position. Test with the 5 letter examples above. There are several ways to attack this. One way is to use multisets [(Counter)](https://docs.python.org/3.5/library/collections.html#collections.Counter) from the collections module, another is to use permutations from [itertools](https://docs.python.org/3/library/itertools.html?highlight=permutations#itertools.permutations). Of the two, I \*\*\_highly*\*\* recommend itertools. The multiset implementation is far more difficult to get correct.

        Loaded words_dat.txt containing 5757 five-letter English words.
        Two words are connected if they differ in one letter.
        Graph has 5757 nodes with 112278 edges
        16 connected components
        Shortest path between chaos and order is
        chaos
        chose
        chore
        coder
        order
        Shortest path between nodes and graph is
        nodes
        anode
        agone
        anger
        gaper
        graph
        Shortest path between moron and smart is
        moron
        manor
        roams
        smart
        Shortest path between flies and swims is
        flies
        isles
        semis
        swims

        import gzip
        from string import ascii_lowercase as lowercase

        import networkx as nx
        import itertools as it

        #-------------------------------------------------------------------
        #   The Words/Ladder graph of Section 1.1
        #-------------------------------------------------------------------


        def generate_graph(words):
            G = nx.Graph(name="words")
            lookup = dict((c, lowercase.index(c)) for c in lowercase)

            def edit_distance_one(word):
                word_permutations = it.permutations(word)
                for word_perm in word_permutations:
                    word_p = ''
                    for c in word_perm:
                        word_p += c
                    for i in range(len(word_p)):
                        left, c, right = word_p[0:i], word_p[i], word_p[i + 1:]
                        j = lookup[c]  # lowercase.index(c)
                        for cc in lowercase[j + 1:]:
                            yield left + cc + right
            candgen = ((word, cand) for word in sorted(words)
                    for cand in edit_distance_one(word) if cand in words)
            G.add_nodes_from(words)
            for word, cand in candgen:
                G.add_edge(word, cand)
            return G


        def words_graph():
            """Return the words example graph from the Stanford GraphBase"""
            fh = gzip.open('words_dat.txt.gz', 'r')
            words = set()
            for line in fh.readlines():
                line = line.decode()
                if line.startswith('*'):
                    continue
                w = str(line[0:5])
                words.add(w)
            return generate_graph(words)


        if __name__ == '__main__':
            G = words_graph()
            print("Loaded words_dat.txt containing 5757 five-letter English words.")
            print("Two words are connected if they differ in one letter.")
            print("Graph has %d nodes with %d edges"
                % (nx.number_of_nodes(G), nx.number_of_edges(G)))
            print("%d connected components" % nx.number_connected_components(G))

            for (source, target) in [('chaos', 'order'),
                                    ('nodes', 'graph'),
                                    ('moron', 'smart'),
                                    ('flies', 'swims')]:
                print("Shortest path between %s and %s is" % (source, target))
                try:
                    sp = nx.shortest_path(G, source, target)
                    for n in sp:
                        print(n)
                except nx.NetworkXNoPath:
                    print("None")

<!--
- Then create/fork a github repository for your project and work on your first commit
-->
