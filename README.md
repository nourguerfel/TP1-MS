# TP1-MS
### Guerfel Nour ,MPI1
# TP1 — Inversion de Contrôle (IoC) et Injection de Dépendances (DI)

> **Matière :** Architecture MicroServices  
> **Environnement :** Java 17 · IntelliJ IDEA · Maven · Spring Framework  
> **Compte rendu :** activités 1-1 à 1-4, observations, réponses aux questions et captures d'écran personnelles.

## Sommaire

1. [Présentation et objectifs](#1-présentation-et-objectifs)
2. [Vérification du JDK](#2-vérification-du-jdk)
3. [Création et structure du projet Maven](#3-création-et-structure-du-projet-maven)
4. [Activité 1-1 — Injection statique](#4-activité-1-1--injection-statique)
5. [Activité 1-2 — Injection dynamique](#5-activité-1-2--injection-dynamique)
6. [Activité 1-3 — Injection avec Spring XML](#6-activité-1-3--injection-avec-spring-xml)
7. [Activité 1-4 — Injection avec Spring et annotations](#7-activité-1-4--injection-avec-spring-et-annotations)
8. [Comparaison des approches](#8-comparaison-des-approches)
9. [Conclusion générale](#9-conclusion-générale)

---

## 1. Présentation et objectifs

Ce travail pratique étudie le **couplage faible**, l'**injection de dépendances** (*Dependency Injection*, DI) et l'**inversion de contrôle** (*Inversion of Control*, IoC) à travers quatre modes d'assemblage des objets : instanciation statique, instanciation dynamique par réflexion, conteneur Spring configuré en XML et conteneur Spring configuré par annotations.

### Objectifs généraux

- Comprendre les concepts d'injection de dépendances et d'inversion de contrôle.
- Découvrir le couplage faible à travers la programmation par interfaces.
- Comprendre l'utilisation d'un fichier de configuration XML.
- Comparer plusieurs stratégies de création et de gestion des objets.

### Principe de l'application

L'application utilise trois couches :

- **`dao`** : accès aux données, via `IDao` et `DaoIMP`.
- **`metier`** : traitement métier, via `IMetier` et `MetierIMP`.
- **`presentation`** : lancement de l'application et affichage du résultat.

`MetierIMP` dépend de l'**interface** `IDao`, et non d'une implémentation concrète. Dans l'exemple de la correction, `DaoIMP.getValue()` renvoie `3` et `MetierIMP.calcul()` multiplie cette valeur par `7` : le résultat attendu des quatre activités est **`21.0`**.

```text
Presentation → IMetier ← MetierIMP → IDao ← DaoIMP
```

Cette architecture en couches illustre surtout la **séparation présentation / métier / DAO** ; elle ne constitue pas à elle seule un MVC complet au sens d'une application avec vue et contrôleur distincts.

---

## 2. Vérification du JDK

Avant de commencer, la présence du **JDK 17** a été vérifiée dans PowerShell avec :

```powershell
java -version
javac -version
echo $env:JAVA_HOME
```

- `java -version` vérifie la version de Java disponible pour l'exécution.
- `javac -version` vérifie la version du compilateur Java.
- `echo $env:JAVA_HOME` affiche le répertoire configuré pour le JDK.

Selon la vérification réalisée, les commandes `java -version` et `javac -version` indiquent la version **17.0.20.1**, tandis que `JAVA_HOME` pointe vers le dossier du JDK.

**Figure 1 — Vérification du JDK 17 dans PowerShell**

![Vérification de Java, javac et JAVA_HOME](images/jdk-verification.png)

---

## 3. Création et structure du projet Maven

### 3.1. Création du projet `TP1`

Le projet a été créé dans **IntelliJ IDEA** selon les étapes suivantes :

1. Ouvrir **New Project**.
2. Sélectionner le langage **Java**.
3. Choisir **Maven** comme système de build.
4. Sélectionner le **JDK 17**.
5. Nommer le projet **`TP1`**.
6. Indiquer **GroupId : `org.example`** et **ArtifactId : `TP1`**.
7. Décocher **Add sample code** afin de créer les classes du TP manuellement.

**Figure 2 — Création du projet Maven dans IntelliJ IDEA**

![Création du projet Maven TP1](images/creation-projet-maven.png)

### 3.2. Organisation des fichiers

```text
TP1/
├── pom.xml
├── README.md
├── images/
│   ├── jdk-verification.png
│   ├── creation-projet-maven.png
│   ├── activite1-1.png
│   ├── activite1-2(1).png
│   ├── activite1-2(2).png
│   ├── activite1-3.png
│   ├── activite1-3(2).png
│   └── activite1-4.png
└── src/
    └── main/
        ├── java/
        │   ├── dao/
        │   │   ├── IDao.java
        │   │   └── DaoIMP.java
        │   ├── metier/
        │   │   ├── IMetier.java
        │   │   └── MetierIMP.java
        │   └── presentation/
        │       ├── PresentationStatique.java
        │       ├── PresentationDynamique.java
        │       ├── PresentationAvecSpring.java
        │       └── PresentationAvecAnnotation.java
        └── resources/
            ├── config.txt
            └── config.xml
```

> **Remarque :** dans un projet Maven, les fichiers de configuration sont placés dans `src/main/resources`, ce qui permet de les rendre accessibles dans le *classpath*.

### 3.3. Interfaces et classes communes

Les deux interfaces définissent les contrats utilisés par les couches de l'application :

**`dao/IDao.java`**

```java
package dao;

public interface IDao {
    double getValue();
}
```

**`metier/IMetier.java`**

```java
package metier;

public interface IMetier {
    double calcul();
}
```

**`dao/DaoIMP.java` — logique DAO**

```java
package dao;

public class DaoIMP implements IDao {
    @Override
    public double getValue() {
        return 3;
    }
}
```

**`metier/MetierIMP.java` — logique métier**

```java
package metier;

import dao.IDao;

public class MetierIMP implements IMetier {
    private IDao dao;

    public void setDao(IDao dao) {
        this.dao = dao;
    }

    @Override
    public double calcul() {
        return dao.getValue() * 7;
    }
}
```

> Les annotations Spring seront ajoutées aux classes lors de l'activité 1-4. Les extraits ci-dessus présentent d'abord leur fonctionnement sans annotations.

---

## 4. Activité 1-1 — Injection statique

### Objectif

Explorer le **couplage faible** en utilisant l'injection de dépendances par **instanciation statique**.

### Travail réalisé

Les packages `presentation`, `metier` et `dao` ont été créés, ainsi que les interfaces et les classes demandées. La classe `PresentationStatique` instancie directement `DaoIMP` et `MetierIMP`, puis associe les objets avec `setDao()`.

**`presentation/PresentationStatique.java`**

```java
package presentation;

import dao.IDao;
import dao.DaoIMP;
import metier.MetierIMP;

public class PresentationStatique {
    public static void main(String[] args) {
        IDao dao = new DaoIMP();
        MetierIMP metier = new MetierIMP();
        metier.setDao(dao);
        System.out.println(metier.calcul());
    }
}
```

### Résultat de l'exécution

```text
21.0
```

![Activité 1-1 — code et résultat](images/activite1-1.png)

### Question 9 — Comment cette solution illustre-t-elle le couplage faible ?

La classe `MetierIMP` déclare une dépendance de type `IDao` plutôt que `DaoIMP`. Elle ne connaît donc que le **contrat** de la couche DAO. Il devient possible d'injecter une autre classe implémentant `IDao`, sans modifier la méthode `calcul()`. Le couplage reste toutefois présent dans la classe de présentation, qui choisit explicitement l'implémentation concrète.

### Question 10 — Le principe d'inversion de contrôle est-il respecté ? Justifier.

**Pas au niveau de la création et de l'assemblage des objets par un conteneur.** La classe de présentation utilise `new` et appelle `setDao()` elle-même : elle conserve le contrôle de l'instanciation. On applique bien le principe de **dépendance injectée depuis l'extérieur de `MetierIMP`**, mais pas encore une inversion de contrôle confiée à un mécanisme externe tel que Spring.

### Question 11 — Que modifier pour éviter que la présentation crée directement les objets ?

Il faut transférer cette responsabilité vers un mécanisme d'assemblage distinct : par exemple une **configuration externe** associée à la réflexion Java (activité 1-2), puis un **conteneur IoC** capable de créer les objets et d'injecter leurs dépendances (activités 1-3 et 1-4).

---

## 5. Activité 1-2 — Injection dynamique

### Objectif

Explorer l'injection de dépendances à partir de **noms de classes lus dans un fichier de configuration**, sans écrire directement leurs constructeurs dans la classe de présentation.

### Travail réalisé

Le fichier **`src/main/resources/config.txt`** contient :

```text
dao.DaoIMP
metier.MetierIMP
```

La classe `PresentationDynamique` lit ces lignes, charge les classes avec `Class.forName()`, crée les objets par réflexion et injecte le DAO dans l'objet métier.

**Extrait représentatif de `PresentationDynamique.java`**

```java
String daoClassName = /* première ligne lue depuis config.txt */;
String metierClassName = /* deuxième ligne lue depuis config.txt */;

IDao dao = (IDao) Class.forName(daoClassName)
        .getDeclaredConstructor().newInstance();
IMetier metier = (IMetier) Class.forName(metierClassName)
        .getDeclaredConstructor().newInstance();

metier.getClass().getMethod("setDao", IDao.class).invoke(metier, dao);
System.out.println(metier.calcul());
```

Cet extrait illustre les opérations centrales ; la classe complète doit également contenir la lecture du fichier et la gestion des exceptions.

### Résultat de l'exécution

```text
21.0
```

**Captures d'écran — configuration et exécution**

![Activité 1-2 — configuration et code](images/activite1-2(1).png)

![Activité 1-2 — exécution](images/activite1-2(2).png)

### Question 7 — Pourquoi s'agit-il d'une injection par instanciation dynamique ?

Le code de présentation ne désigne plus directement les implémentations avec `new DaoIMP()` et `new MetierIMP()`. Leur **nom complet** est fourni par `config.txt` ; `Class.forName()` permet de retrouver les classes à l'exécution et la réflexion permet de créer leurs instances et d'appeler le setter d'injection. Le choix de l'implémentation est ainsi reporté dans la configuration.

### Question 8 — Pourquoi cette approche se rapproche-t-elle davantage de l'IoC ?

Le **choix des classes concrètes** n'est plus fixé dans le code de la présentation : il est externalisé dans un fichier. Il devient possible de changer l'implémentation indiquée dans `config.txt`, à condition que la nouvelle classe existe, respecte les interfaces attendues et soit instanciable. Toutefois, `PresentationDynamique` lit toujours la configuration et pilote elle-même la réflexion et l'injection : elle ne bénéficie pas encore d'un véritable conteneur IoC.

### Question 9 — Avantages et limites

**Avantages :**

- Changement d'implémentation possible via le fichier de configuration.
- Diminution du couplage avec les classes concrètes dans le code de présentation.
- Découverte du chargement dynamique et de l'API de réflexion Java.

**Limites :**

- Gestion manuelle des erreurs, des constructeurs et de l'injection.
- Erreurs de noms de classes ou de méthodes détectées généralement **à l'exécution**.
- Code de réflexion plus difficile à maintenir et à déboguer.
- Pas de gestion automatisée des dépendances complexes ni du cycle de vie des objets.

---

## 6. Activité 1-3 — Injection avec Spring XML

### Objectif

Utiliser **Spring** et un **fichier XML** pour confier au conteneur la création des objets et l'injection de leurs dépendances.

### Travail réalisé

**Dépendances Maven** nécessaires dans `pom.xml` :

```xml
<properties>
    <maven.compiler.source>17</maven.compiler.source>
    <maven.compiler.target>17</maven.compiler.target>
    <spring.version>6.2.8</spring.version>
</properties>

<dependencies>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-core</artifactId>
        <version>${spring.version}</version>
    </dependency>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-context</artifactId>
        <version>${spring.version}</version>
    </dependency>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-beans</artifactId>
        <version>${spring.version}</version>
    </dependency>
</dependencies>
```

> Il s'agit d'un **extrait** du fichier `pom.xml`, à intégrer dans l'élément `<project>` de Maven. La version indiquée reprend la correction fournie.

**`src/main/resources/config.xml`**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
                           https://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="dao" class="dao.DaoIMP" />

    <bean id="metier" class="metier.MetierIMP">
        <property name="dao" ref="dao" />
    </bean>
</beans>
```

- `<bean>` déclare les objets gérés par Spring.
- `<property name="dao" ref="dao" />` demande au conteneur d'injecter le bean `dao` dans le bean `metier` par le setter `setDao()`.

**`presentation/PresentationAvecSpring.java`**

```java
package presentation;

import metier.IMetier;
import org.springframework.context.support.ClassPathXmlApplicationContext;

public class PresentationAvecSpring {
    public static void main(String[] args) {
        try (ClassPathXmlApplicationContext context =
                     new ClassPathXmlApplicationContext("config.xml")) {
            IMetier metier = context.getBean("metier", IMetier.class);
            System.out.println(metier.calcul());
        }
    }
}
```

### Résultat de l'exécution

```text
21.0
```

**Captures d'écran — configuration XML et résultat**

![Activité 1-3 — configuration et code](images/activite1-3.png)

![Activité 1-3 — exécution](images/activite1-3(2).png)

### Question 6 — Comparaison avec l'activité 1-2 concernant la gestion des dépendances

Dans l'activité 1-2, la présentation doit elle-même lire le fichier, utiliser la réflexion, instancier les objets et appeler la méthode d'injection. Avec **Spring XML**, le fichier `config.xml` **décrit** les beans et leurs relations ; le **conteneur Spring** crée ces objets et applique leurs dépendances. Le code de présentation se limite essentiellement à initialiser le contexte et récupérer le bean métier.

### Question 7 — Différence entre création manuelle, réflexion et Spring

| Méthode | Création et assemblage | Caractéristique principale |
|---|---|---|
| Création manuelle | `new` et appel explicite à `setDao()` | Choix des classes dans le code de présentation. |
| Réflexion | `Class.forName()`, constructeur par réflexion, appel dynamique de `setDao()` | Choix des classes depuis un fichier ; gestion de l'injection encore manuelle. |
| Conteneur Spring | Définition des beans XML, puis `ApplicationContext` | Création et injection confiées à Spring. |

Ainsi, Spring centralise et automatise les tâches techniques d'assemblage qui étaient effectuées dans le code lors des activités précédentes.

---

## 7. Activité 1-4 — Injection avec Spring et annotations

### Objectif

Découvrir les annotations **`@Repository`**, **`@Service`** et **`@Autowired`** pour déclarer et injecter les composants sans utiliser de déclaration XML des beans.

### Travail réalisé

**`dao/DaoIMP.java`**

```java
package dao;

import org.springframework.stereotype.Repository;

@Repository
public class DaoIMP implements IDao {
    @Override
    public double getValue() {
        return 3;
    }
}
```

**`metier/MetierIMP.java`**

```java
package metier;

import dao.IDao;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

@Service
public class MetierIMP implements IMetier {
    @Autowired
    private IDao dao;

    public void setDao(IDao dao) {
        this.dao = dao;
    }

    @Override
    public double calcul() {
        return dao.getValue() * 7;
    }
}
```

Le setter est **conservé** pour la compatibilité avec les autres activités ; dans l'activité 1-4, Spring réalise l'injection via `@Autowired`.

**`presentation/PresentationAvecAnnotation.java`**

```java
package presentation;

import metier.IMetier;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;

public class PresentationAvecAnnotation {
    public static void main(String[] args) {
        try (AnnotationConfigApplicationContext context =
                     new AnnotationConfigApplicationContext("dao", "metier")) {
            IMetier metier = context.getBean(IMetier.class);
            System.out.println(metier.calcul());
        }
    }
}
```

### Résultat de l'exécution

```text
21.0
```

![Activité 1-4 — annotations Spring et exécution](images/activite1-4.png)

### Question 5 — Comment Spring gère-t-il l'injection avec les annotations ?

1. `AnnotationConfigApplicationContext("dao", "metier")` **scanne** les packages concernés.
2. `@Repository` indique que `DaoIMP` est un composant de la couche d'accès aux données.
3. `@Service` indique que `MetierIMP` est un composant métier.
4. Spring crée des **beans** pour les classes détectées.
5. `@Autowired` sur le champ de type `IDao` demande à Spring de résoudre cette dépendance en injectant un bean compatible, ici `DaoIMP`.
6. La présentation récupère le bean métier prêt à être utilisé.

Si plusieurs beans compatibles avec `IDao` existent, il faut résoudre l'ambiguïté, par exemple avec `@Qualifier` ou `@Primary`.

### Question 6 — Pourquoi cette solution est-elle plus maintenable que la configuration XML ?

Pour ce TP, les annotations simplifient la configuration : les rôles des classes sont visibles directement dans le code, tandis que le **scan** détecte automatiquement les composants. Le nombre de déclarations XML diminue, ce qui facilite la lecture et les modifications courantes. Cela ne signifie pas que XML est toujours moins adapté : il peut être utile lorsqu'on veut conserver une configuration totalement externe aux classes Java.

### Question 7 — En quoi cette activité représente-t-elle une application complète du principe IoC ?

La **création des objets**, leur **gestion comme beans** et la **résolution des dépendances** sont assurées par le conteneur Spring. La présentation ne décide plus elle-même de la construction des instances DAO et métier et ne réalise plus l'injection. Le contrôle de l'assemblage est donc effectivement **inversé** : il passe du code applicatif vers Spring.

---

## 8. Comparaison des approches

| Activité | Choix des classes | Création des objets | Injection des dépendances | Fichier ou mécanisme principal |
|---|---|---|---|---|
| **1-1 Statique** | Dans le code | Programmeur (`new`) | Manuelle (`setDao`) | Java |
| **1-2 Dynamique** | Dans un fichier | Réflexion pilotée par le programme | Manuelle, par réflexion | `config.txt`, `Class.forName()` |
| **1-3 Spring XML** | Dans le XML | Conteneur Spring | Conteneur Spring | `config.xml`, `<bean>`, `<property>` |
| **1-4 Spring annotations** | Via annotations et scan | Conteneur Spring | Conteneur Spring | `@Repository`, `@Service`, `@Autowired` |

**Observation :** le **couplage faible** dépend surtout du fait que la couche métier s'appuie sur `IDao` au lieu de `DaoIMP`. Les quatre activités conservent ce principe, mais elles font évoluer **la responsabilité de la création et de l'assemblage** des objets. Spring réduit le code de configuration et d'injection écrit à la main.

### Récapitulatif des résultats attendus

| Classe exécutée | Résultat |
|---|---:|
| `PresentationStatique` | `21.0` |
| `PresentationDynamique` | `21.0` |
| `PresentationAvecSpring` | `21.0` |
| `PresentationAvecAnnotation` | `21.0` |

---

## 9. Conclusion générale

Ce TP montre une évolution progressive de la gestion des dépendances :

1. **L'instanciation statique** introduit l'injection par setter et met en évidence le couplage faible obtenu grâce à l'interface `IDao`.
2. **L'instanciation dynamique** externalise le choix des classes à travers un fichier de configuration et utilise la réflexion Java.
3. **Spring XML** confie au conteneur la création des beans et la résolution des dépendances déclarées en XML.
4. **Spring avec annotations** automatise la découverte des composants et leur injection, sans configuration XML des beans.

La distinction essentielle est la suivante : le **couplage faible** réduit la dépendance aux implémentations concrètes ; l'**injection de dépendances** consiste à fournir les objets nécessaires depuis l'extérieur ; l'**inversion de contrôle** confie la responsabilité de leur création et de leur assemblage à un mécanisme externe, ici le conteneur Spring.

---

### Référence pédagogique

- [Correction du TP1 proposée par l'enseignante](https://github.com/amal-abid05/TP1-MS)

> **Note sur les figures :** les liens `images/...` correspondent aux captures d'écran du projet personnel. Ils fonctionneront sur GitHub uniquement si ces fichiers ont été ajoutés au dépôt sous les mêmes noms et avec la même casse.

