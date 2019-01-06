# Une première application

Pour mettre en pratique l'utilisation d'Android Studio et prendre
en main un projet Android, nous allons créer une application simulant
un lancer de pièce (pile ou face).

[Résumé d'introduction](../1_ressources/3_premiere_application.pdf)

## Objectifs

Il y deux objectifs fondamentaux dans cette partie :
* revoir l'utilisation des objets en Java : Déclaration, Instanciation, Utilisation
* introduire le concept d'écoute événementielle.

## Le principe de l'application

L'application sera composée d'une vue permettant d'afficher du texte (`TextView`).
Il y aura, stockés dans les ressources, deux labels correspondants aux résultats
en anglais ("Tail" ou "Head").

Au démarrage de l'application, le programme génère un booléen aléatoirement égal à `true` ou `false`, et
affiche le label correspondant. L'utilisateur peut effectuer un nouveau lancer en cliquant sur un bouton.

## Mise en place

Créez un nouveau projet appelé "TwoSidesOfTheCoin" :

![Écran de création 1/4](screens/1_creation_1.png)

![Écran de création 2/4](screens/1_creation_2.png)

![Écran de création 3/4](screens/1_creation_3.png)

![Écran de création 4/4](screens/1_creation_4.png)

Regardez la structure du projet créé par Android Studio :

![Structure du projet](screens/2_environnement_1_structure.png)

Ouvrez les fichiers `manifest`, `CoinActivity`, et `activity_coin` en mode Design et Text :

![Structure du projet](screens/2_environnement_2_manifest.png)

![Structure du projet](screens/2_environnement_3_CoinActivity.png)

![Structure du projet](screens/2_environnement_4_activity_coin_design.png)

![Structure du projet](screens/2_environnement_5_activity_coin_text.png)


## Pas à pas des modifications

### 1) Modification de la vue

Dans un premier temps, il faut modifier la vue pour ajouter une zone de texte.
Modifier le `TextView` automatiquement ajouté par Android Studio : renommez le en `tv_coin_results`, agrandissez-le pour qu'il prenne tout l'espace et videz le texte qu'il contient.

Vous pouvez effectuer cela en utilisant le mode *Design* qui permet d'éditer facilement les composants graphiques et de modifier les propriétés via le menu de droite. Cependant cette interface est limitée, et il est parfois plus rapide d'utiliser le mode *Text*.

Vous devez obtenir le code suivant :

```xml
    <TextView
        android:id="@+id/tv_coin_result"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_alignParentRight="true"
        android:layout_alignParentEnd="true"
        android:layout_alignParentLeft="true"
        android:layout_alignParentStart="true"
        android:layout_alignParentBottom="true" />
```

### 2) Ajout des labels dans les fichiers strings

Ouvrez le fichier `res/values/string.xml`. Vous pouvez voir qu'il y a déjà une ressource nommée `app_name` ayant pour valeur « TwoSidesOfTheCoin ».

Il faut ajouter deux ressources nommées `tail` et `head` avec pour valeur respective « Tail » et « Head ».

### 3) Modification de l'activité

Nous allons maintenant modifier l'activité (fichier `java/[nom de votre package]/CoinActivity`). Pour rapel la méthode `onCreate` est exécutée à la création de l'activité. Comme cette activité est l'activité de démarrage de l'application, cette fonction sera exécutée sans intervention de l'utilisateur.

Vous pouvez voir qu'il y a deux lignes dans la fonction, elles permettent d'afficher la vue. Vous devez effectuer vos modifications après.

On va ajouter une méthode à la classe, nommée `coinFlip` qui va simuler le lancement de la pièce, puis afficher « Tail » ou « Head » dans le `TextView`.

#### a) Lancement de la pièce

```java
// Voici comment générer aléatoirement un booléen en java :
Random randomGenerator = new Random();
boolean random = randomGenerator.nextBoolean();

// Voici comment récupérer les labels créés précédemment :
String tail = getResources().getString(R.string.tail);
String head = getResources().getString(R.string.head);

// String result = tail ou head ?
```


#### b) Afficher le résultat

Il faut dans un second temps récupérer le `TextView` de la vue, de manière à instancier un objet Java qui le représente.

Cela s'effectue grâce à la fonction `findViewById([ressource_id])` qui prend comme paramètre l'identifiant de votre contrôle graphique (qui est une ressource). Si votre élément s'appelle `tv_coin_result`, l'identifiant de ressource est : `R.id.tv_coin_result`.

Il faut donc déclarer votre variable (comme attribut de classe) :

```java
    TextView tv_coinResult;
```

Puis instancier l'élément (dans la fonction `onCreate`) :

```java
    tv_coinResult = (TextView)findViewById(R.id.tv_coin_result);
```

Enfin il suffit de définir le texte de notre `TextView` (dans la méthode `coinFlip`) :

```java
    tv_coinResult.setText(result);
```


Voici la fonction `coinFlip` à obtenir :

```java
    public void coinFlip() {
        // Creation of a randomGenerator
        Random randomGenerator = new Random();

        // Tail or not Tail that is the question?
        boolean tail = randomGenerator.nextBoolean();

        // Put the result in a string
        String result;
        if (tail) {
            result = getResources().getString(R.string.tail);
        } else {
            result = getResources().getString(R.string.head);
        }

        // Set the result
        tv_coinResult.setText(result);
    }
```

Il faut en suite d'appeler cette fonction dans la fonction `onCreate` (après avoir instancier le `TextView`) :

```java
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_coin);

        // I - Instanciation les objets Java représentant les composants graphiques
        tv_coinResult = (TextView)findViewById(R.id.tv_coin_result);

        // III - Autre traitement à effectuer au début...
        coinFlip();
    }
```

#### Le point à retenir

La ligne la plus importante est :

```java
        // I - Instanciation les objets Java représentant les composants graphiques
        tv_coinResult = (TextView)findViewById(R.id.tv_coin_result);
```

Cette ligne permet d'instancier en Java l'objet `TextView` correspondant au code XML écrit à l'étape **1)**.
La fonction `findViewById` de signature `View findViewById(String id)` est fournie par l'API d'Android. Elle permet de récupérer n'importe quel élément d'une vue et d'en faire un objet Java. Cela permet de pouvoir agir sur l'objet via une fonction Java.

La fonction renvoie un objet `View`. Or tous les éléments que l'on peut afficher héritent de l'objet `View`.
On doit donc *caster* l'objet (en ajoutant le `(TextView)`), ce qui revient à préciser à Java que l'objet est un `TextView` et non autre chose (`EditText`, `Button`, ...).

Extrait du diagramme UML des composants graphiques :

![Extrait du diagramme UML des composants graphiques](uml/views.png "Extrait du diagramme UML des composants graphiques")


#### Remarque sur les imports :

Remarquez les `import` à effectuer :

```java
import android.support.v7.app.AppCompatActivity;
import android.os.Bundle;
import android.widget.TextView;

import java.util.Random;
```

Vous utilisez des librairies, il faut donc les importer (Android Studio arrive très bien à vous proposer d'ajouter automatiquement les librairies les plus courantes, utilisez le `Alt+Entrer`).

### 4) Lancement de l'application

Connectez votre téléphone à l'ordinateur et lancez le programme dans Android Studio (`Maj+F10`).
Android Studio ouvre une fenêtre : sélectionnez votre mobile. L'application doit se lancer sur votre téléphone !

Vous aurez peut-être besoin d'installer les pilotes ADB : [Voir ce document](Installation des pilotes ADB.pdf)

### 5) Ajout du bouton

Votre programme fonctionne ! On va maintenant ajouter un bouton pour que l'utilisateur puisse relancer la pièce !

Ajouter un bouton dans la vue, en haut, nommez-le `b_coin_flip`. Personnalisez le label du bouton en utilisant une ressource...

Pour que tout s'organise bien, positionnez le `Button` et le `TextView` dans un `LinearLayout`  à l'orientation verticale.

Voici le code XML obtenu :

```xml
<?xml version="1.0" encoding="utf-8"?>
<android.support.constraint.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".CoinActivity">

    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:layout_marginTop="8dp"
        android:layout_marginBottom="8dp"
        android:orientation="vertical">

        <Button
            android:id="@+id/b_coin_flip"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:text="@string/flip" />

        <TextView
            android:id="@+id/tv_coin_result"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content" />
    </LinearLayout>

</android.support.constraint.ConstraintLayout>
```

Nous voulons que Java génère un événement lorsque l'utilisateur clique sur le bouton.
Pour cela, nous devons ajouter un *event listener* (écouteur d'événement) au bouton.

Il va falloir instancier le `Button` en objet Java (comme nous l'avons fait pour le `TextView`) :
1. ajoutez une déclaration en attribut de classe pour stocker le bouton ;
2. instancier le bouton dans la méthode `onCreate`.

Maintenant, nous devons ajouter l'écouteur d'événement. Nous allons utiliser la fonction `setOnClickListener` :

```java
    // II - Ajout des écouteurs d'événements aux composants graphiques représentés par des objets Java

    // Ajout d'un écouteur d'événement "OnClickListener" anonyme à l'objet "Button" b_coin_flip représentant le composant "Button" "b_coin_flip"
    b_coin_flip.setOnClickListener(new View.OnClickListener() {
        @Override
        public void onClick(View v) {
            // Here we have the function that flip the coin
            CoinActivity.this.coinFlip();
        }
    });
```

Dans ce code, est instancié un écouteur d'événement anonyme (non stocké dans une variable) et directement définit comme l'écouteur d'événement du bouton.

La fonction `onClick` de cet écouteur est appelée au clic sur le bouton.

On fait alors appel à la fonction qu'on a crée précédement, attention, nous ne sommes plus dans la classe `CoinActivity`, il ne faut pas faire simplement `this.coinFlip()`, mais `CoinActivity.this.coinFlip()`.

Vous pouvez modifier le programme pour *ajouter* le dernier lancé au `TextView`... (Il faudra lors ajouter un `ScrollView` autour du `TextView`...)

Voici la structure globale attendue :

```java
public class CoinActivity extends AppCompatActivity {

    /**
     * Déclaration des objets Java représentant les composants graphiques
     * (en attributs de la classe, ils sont donc accessibles depuis toutes les méthodes)
     */
    TextView tv_coinResult;
    Button b_coin_flip;

    /**
     * Fonction exécutée à la création de la vue
     * Elle DOIT instancier les objets Java représentant les composants
     * graphiques ET leur ajouter des écouteurs d'événements si besoin.
     */
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_coin);

        // I - Instanciation les objets Java représentant les composants graphiques
        tv_coinResult = (TextView)findViewById(R.id.tv_coin_result);
        b_coin_flip = (Button)findViewById(R.id.b_coin_flip);

        // II - Ajout des écouteurs d'événements aux composants graphiques représentés par des objets Java

        // Ajout d'un écouteur d'événement "OnClickListener" anonyme à l'objet "Button" b_coin_flip représentant le composant "Button" "b_coin_flip"
        b_coin_flip.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                // Here we have the function that flip the coin
                CoinActivity.this.coinFlip();
            }
        });

        // III - Autre traitement à effectuer au début...
        coinFlip();
    }

    protected  void coinFlip() {
        // Creation of a randomGenerator
        Random randomGenerator = new Random();

        // Tail or not Tail that is the question?
        boolean tail = randomGenerator.nextBoolean();

        // Put the result in a string
        String result;
        if (tail) {
            result = getResources().getString(R.string.tail);
        } else {
            result = getResources().getString(R.string.head);
        }

        // Set the result
        tv_coinResult.setText(result);
    }
}
```

## Les points à retenir

Les **ressources** permettent de gérer des *vues*, des images et des *constantes* (chaînes de caractères et autres).

Les **méthodes** associées aux **activités** permettent de coder des fonctionnalités.

Il est possible d'**accéder aux éléments des vues depuis ces méthodes** pour agir dessus et transmettre des informations aux utilisateurs.

Beaucoup de **composants graphiques** existent déjà pour vous aider à développer des interfaces riches sans efforts.

Pour **agir sur les composants graphiques** il faut :
* **instancier des objets Java représentant ces composants** (via `findViewById`) ;
* leur associer des **écouteurs d'événements** (event listerners).

## Organisation du code

Pour ne pas faire d'erreur sur la déclaration et l'instanciation des composants graphique, je vous conseil fortement de :
* déclarer toutes les varaibles de composants comme attributs de classe (vous pourrez simplement accéder à tous les composants dans toutes les fonctions de classe) ;
* instancier tous les composants dès le début de la fonction `onCreate`;
* associer les écouteurs d'événement aux composants juste après (créez sur votre classe d'activité une méthode à éxécuter pour chaque écouteur d'événement).

## Projet complet

Vous pouvez retrouver le projet complet ici : [https://gitlab.com/vsasyan/TwoSidesOfTheCoin](https://gitlab.com/vsasyan/TwoSidesOfTheCoin)
