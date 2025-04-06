# Beziehungen in Quarkus
Kurze generelle beziehungs erklärung. ACHTUNG PSEUDOCODE des is aus simplistic gründen kein valides java

wenn du

```JAVA
Movie{
    name

    @OneToMany
    List<Actor> actors
}

Actor{
    String name
}
```

dann macht quarkus folgende Tabellen

```
actor(id, name, movie_id)
movie(id, name)
```

also ein Schauspieler KANN NUR in einem einzigen Film mitspielen

wenn du

```JAVA
Movie{
    String name

    @ManyToMany
    List<Actor> actors
}

Actor{ 
    String name
}
```

dann macht quarkus folgende Tabellen

```
actor(id, name)
movie_actor(actor_id, movie_id)
movie(id, name)
```

also ein Schauspieler kann in mehreren Filmen mitspielen

In beiden Fällen können wir innerhalb von java aber nur über den Film eine Liste von seinen Schauspielern sehen.

Wir können also

```JAVA
Movie{
    String name

    @OneToMany
    List<Actor> actors
}

Actor{
    String name

    @ManyToOne
    Movie movie
}
```

dann macht quarkus folgende Tabellen

```
actor(id, name, movie_id)
movie(id, name)
```

also der Schauspieler immer noch nur in einem einzigen Film Spielen. ABER diesmal können wir auch über den schauspieler innerhalb von java auf dessen einen Film zugreifen.

Und dann können wir noch

```JAVA
Movie{
    String name

    @ManyToMany
    List<Actor> actors
}

Actor{
    String name

    @ManyToMany(mappedBy = "actors")
    List<Movie> movies
}
```

folgende Tabellen werden erstellt:

```
actor(id, name)
movie_actor(actor_id, movie_id)
movie(id, name)
```

Also jetzt können wir sowohl von "Movie" eine liste der "actors" als auch von "Actor" eine liste der "movies" über java abfragen.

## mappedBy
Bei einem mapped by geht es immer nur darum, dass wir ein mapping der daten in einer java variable abbilden. Dabei wird nie eine neue Tabelle erstellt sondern nur die existierenden Tabellen ausgelesen und in den enties abgebildet. Der string der mitgegeben wird ist der name der variable im *anderen entity* auf das gemapped wird. (siehe Beispiel oben)

## fetch
Sagt quarkus *wann die daten geholt werden*, braucht man im prinzip nur machen wenn man fehler beim request von daten bekommt das zB eine liste nicht initialisiert oder null ist oder das daten nicht geladen werden konnten. Dann kann man mit `fetch = FetchType.EAGER` quarkus zwingen das er soffort *alle* Daten lädt.

## cascade
Sagt dem datenbank system was es mit kindern tun soll wenn im parent daten verändert werden. Per default wird nichts getan, sprich wenn ein Film gelöscht wird, bleibt der Schauspieler bestehen. Man kann das aber mit `CascadeType.ALL` ändern, dass "alle operationen" auch bei den Kindern ausgeführt werden, dann führt das löschen eines Movie zum löschen eines Actor.

```JAVA
Movie{
    name

    @OneToMany(cascade = CascadeType.ALL)
    List<Actor> actors
}

Actor{
    String name
}
```

## join columns
Bei einem @JoinColumns geht es immer darum wie dinge in der datenbank passieren. Für uns ist dabei das häufigste der @JoinColum(name="") dabei geht es darum wie das automatisch generierte fremdschlüssel column in der Datenbank genannt wird. EIN JOIN COLUMNS ÄNDERT NICHTS AN DER FUNKTIONALITÄT.

Also am Vorherigen Beispiel

```JAVA
Movie{
    name

    @OneToMany
    @JoinColumn(name = "film")
    List<Actor> actors
}

Actor{
    String name
}
```

Resultiert in folgenden Datenbank tabellen

```
actor(id, name, film)
movie(id, name)
```

Jetzt wird der Fremdschlüssel (die Movie id) in einer datenbankspalte names "film" gespeichert.

## join table
Ahnlich wie beim join columns gehts beim @JoinTable um darum wie genau die assoziations tabelle für eine @ManyToMany in der Datenbank angelegt wird. Beim "name" gehts um den namen der assoziations tabelle mit joinColumn und inverseJoinColumn können die namen der einzelnnen columns in der assoziationstabelle festgelegt werden. EIN JOIN TABLE ÄNDERT NICHTS AN DER FUNKTIONALITÄT.

Also am Vorherigen Beispiel

```JAVA
Movie{
    String name

    @ManyToMany
		@JoinTable(name = "film_und_schauspieler", joinColumns = @JoinColumn(name = "film"), inverseJoinColumns = @JoinColumn(name = "schauspieler"))
    List<Actor> actors
}

Actor{
    String name

    @ManyToMany(mappedBy = "actors")
    List<Movie> movies
}
```

folgende tabellen werden erstellt:

```
actor(id, name)
film_und_schauspieler(schauspieler, film)
movie(id, name)
```

Jetzt heißt die assoziationstabelle "film_und_schauspieler" und speichert die id des Movie in der Spalte "film" und die id des Actor in der Spalte "schauspieler".