# Google Services

Google fournis énormément d'outils pour enrichir son écosystème. Nous allons voir ceux utiles en géomatique.

## Objectifs

Nous allons utiliser :
* GoogleServices pour se géolocaliser ;
* GoogleMaps pour afficher une carte ;
* les géo-codage de Google pour récupérer une adresse à partir d'une position GPS.

## Le principe de l'application

Nous allons créer une application permettant à utilisateur :
* d'afficher une carte ;
* de se géolocaliser et connaître son adresse actuelle.

## Mise en place

Comme pour votre première application, créez un nouveau projet nommé `PositionName`.

Choisissez cette fois une Activity de base de type « **Google Maps Activity** » :

![Activity de type « Google Maps Activity »](screens/1_creation_1.png)

Et nommez-la `PositionName` :

![PositionName](screens/1_creation_2.png)

## Pas à pas des modifications

L'application devient vite touffue... Pensez à regarder le [diagramme UML](uml/PositionName.png) de l'application pour avoir un résumé des signatures des fonctions à implémenter et le [diagramme de séquence](uml/sequence.png) pour mieux suivre l'enchaînement des fonctions.

### 1) Clef d'API

A la création de votre projet, Android Studio doit vous ouvrir le fichier `app/manifests/AndroidManifest.xml`. Si ce n'est pas le cas, faites-le.

Ce document vous explique comment récupérer la clef d'API Google pour utiliser les cartes.

Il faut normalement suivre [le lien](https://developers.google.com/maps/documentation/android-sdk/get-api-key) et se laisser guider. Ne faites pas ça.

La clef doit être ajoutée dans le fichier `local.properties`. Vous pouvez le retrouver dans la liste des *Gradle Scripts* tout en bas.

```ini
# ...
MAPS_API_KEY=AIzaSyBRR1tCxqn8PJqtDX1e0mE7___________
```

(La fin de la clef vous sera communiquée en direct.)

Dans le fichier AndroidManifest, on ne fait que référence à la variable déclarée précédemment :

```xml
        <meta-data
            android:name="com.google.android.geo.API_KEY"
            android:value="${MAPS_API_KEY}" />
```

Cela évite de publier la clef (le fichier `local.properties` n'est pas versionné).

Par défaut, votre clef est liée :
* à une certaine application (package Java) ;
* à une certaine machine (empreinte du certificat SHA-1).

Le but est d'empêcher d'autres personnes d'utiliser votre clef.

### 2) Visualisation du code

#### a) Fichier AndroidManifest

Nous allons dans un premier temps nous intéresser au fichier `AndroidManifest` (ici présenté sans les commentaires) :

```xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools">

    <application
        android:allowBackup="true"
        android:dataExtractionRules="@xml/data_extraction_rules"
        android:fullBackupContent="@xml/backup_rules"
        android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name"
        android:supportsRtl="true"
        android:theme="@style/Theme.PositionName"
        tools:targetApi="31">

        <meta-data
            android:name="com.google.android.geo.API_KEY"
            android:value="${MAPS_API_KEY}" />

        <activity
            android:name=".MapsActivity"
            android:exported="true"
            android:label="@string/title_activity_maps">
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />

                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>
    </application>

</manifest>
```

Comme vous pouvez le voir, Android Studio a ajouté un bloc par rapport au projet précédent, il concerne la clef d'API Google :

```xml
        <meta-data
            android:name="com.google.android.geo.API_KEY"
            android:value="${MAPS_API_KEY}" />
```

Nous allons ajouter la déclaration de demande de permission à la positon de l'utilisateur. Avant le bloc `application`, ajoutez :

```xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools">

    <uses-permission android:name="android.permission.ACCESS_FINE_LOCATION" />
    <uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION" />

    <!-- ... -->
</manifest>
```

#### b) Layout

Dans un deuxième temps, regardons le code du fichier `activity_maps.xml` décrivant la vue de notre application :

```xml
<?xml version="1.0" encoding="utf-8"?>
<fragment xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:map="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:id="@+id/map"
    android:name="com.google.android.gms.maps.SupportMapFragment"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MapsActivity" />
```

Comme vous pouvez le voir, il n'y a qu'un fragment, qui est un composant spécial permettant de réutiliser des vues. [Plus d'information.](https://developer.android.com/guide/components/fragments.html)

Cela permet de réutiliser certaines parties d'interfaces graphiques. En l’occurrence, nous réutilisons le composant graphique de carte fourni par l'API.

#### c) Java

Enfin nous allons nous intéresser au fichier `MapsActivity.java` :

```java
package fr.ign.positionname;

import androidx.fragment.app.FragmentActivity;

import android.os.Bundle;

import com.google.android.gms.maps.CameraUpdateFactory;
import com.google.android.gms.maps.GoogleMap;
import com.google.android.gms.maps.OnMapReadyCallback;
import com.google.android.gms.maps.SupportMapFragment;
import com.google.android.gms.maps.model.LatLng;
import com.google.android.gms.maps.model.MarkerOptions;

import fr.ign.positionname.databinding.ActivityMapsBinding;

public class MapsActivity extends FragmentActivity implements OnMapReadyCallback {

    private GoogleMap mMap;
    private ActivityMapsBinding binding;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);

        binding = ActivityMapsBinding.inflate(getLayoutInflater());
        setContentView(binding.getRoot());

        // Obtain the SupportMapFragment and get notified when the map is ready to be used.
        SupportMapFragment mapFragment = (SupportMapFragment) getSupportFragmentManager()
                .findFragmentById(R.id.map);
        mapFragment.getMapAsync(this);
    }

    /**
     * Manipulates the map once available.
     * This callback is triggered when the map is ready to be used.
     * This is where we can add markers or lines, add listeners or move the camera. In this case,
     * we just add a marker near Sydney, Australia.
     * If Google Play services is not installed on the device, the user will be prompted to install
     * it inside the SupportMapFragment. This method will only be triggered once the user has
     * installed Google Play services and returned to the app.
     */
    @Override
    public void onMapReady(GoogleMap googleMap) {
        mMap = googleMap;

        // Add a marker in Sydney and move the camera
        LatLng sydney = new LatLng(-34, 151);
        mMap.addMarker(new MarkerOptions().position(sydney).title("Marker in Sydney"));
        mMap.moveCamera(CameraUpdateFactory.newLatLng(sydney));
    }
}
```


##### onCreate

Dans la fonction onCreate, on peut voir qu'il y a trois nouvelles lignes :

```java
        // Obtain the SupportMapFragment and get notified when the map is ready to be used.
        SupportMapFragment mapFragment = (SupportMapFragment) getSupportFragmentManager()
                .findFragmentById(R.id.map);
        mapFragment.getMapAsync(this);
```

La première et la deuxième servent à récupérer le fragment de la vue et à l'instancier en objet Java de type `SupportMapFragment`.
La troisième permet de lancer le chargement de la carte. Ce chargement n'est pas instantané et va être effectué de manière asynchrone (en arrière plan) afin de ne pas bloquer l'interface utilisateur. Une fois ce chargement effectué, le programme exécutera une fonction.

Le paramètre de la fonction `getMapAsync` doit être de type `OnMapReadyCallback`. Or notre classe java est une activité...

##### implements

Si on regarde la ligne de déclaration :

```java
public class MapsActivity extends FragmentActivity implements OnMapReadyCallback {...}
```

On voit que notre activité implémente l'interface `OnMapReadyCallback`, en faisant un `Ctrl + clic gauche` sur cette interface, vous pouvez voir sa signature :

```java
public interface OnMapReadyCallback {
    void onMapReady(GoogleMap var1);
}
```

En déclarant implémenter cette interface, notre activité **doit** avoir une fonction `onMapReady`, de signature `void onMapReady(GoogleMap var1)`. C'est cette fonction qui sera exécutée quand la carte sera chargée.

On parle ici de **callback**, cela ressemble bien évidement beaucoup à de l'**écoute événementielle** (exécuter une certain fonction lorsque qu'un certain événement se produit).

##### onMapReady

Nous allons enfin nous intéresser à la fonction `onMapReady`. Elle prend en paramètre un objet de type GoogleMap, c'est l'instance Java qui représente la carte affichée à l'utilisateur.

Une fois que cette fonction sera exécutée, nous pourrons interagir avec la carte (ajouter des balises, modifier la position, etc).

Remarquez que la fonction récupère l'objet java dans l'attribut de classe `private GoogleMap mMap` déclaré plus haut.

N'importe où dans votre code, lorsque vous avez besoin d'interagir avec la carte, vous **devez** vérifier que votre carte est bien instanciée, c'est à dire que `mMap != null`.

### 3) Modifier la carte

Il y a déjà un code qui affiche une balise sur la carte, modifiez le code pour afficher l'ENSG (48.8410837, 2.5875354) et centrer la carte dessus.

Modifiez le code concernant la `CameraUpdateFactory` pour définir également un zoom, 14 sera bien adapté.

Voici la documentation pour vous aider à trouver la bonne fonction à appeler... : [CameraUpdateFactory](https://developers.google.com/android/reference/com/google/android/gms/maps/CameraUpdateFactory)

Vous pouvez aussi afficher les outils de zoom de la carte : `mMap.getUiSettings().setZoomControlsEnabled(true);` (très utile sur émulateur...).

### 4) Position de l'utilisateur

Nous allons maintenant tenter d'afficher la position de l'utilisateur sur la carte.

La carte fournie par Google est bien évidement déjà capable de le faire.... L'intérêt de le faire par nous même est de pouvoir récupérer les coordonnées GPS de l'utilisateur, pour ensuite trouver son adresse.

Il y a deux manières principales de récupérer la position du client ([documentation](https://developer.android.com/training/location/index.html)) :
* récupérer la dernière position connue ;
* demander un suivi de position.

La première méthode est instantanée, la deuxième va demander de mettre en place un système de callback : il faudra exécuter une certain fonction à chaque mise à jours de position.

En premier lieu, ajoutez une dépendance dans le fichier de configuration de Gradle `Gradle Scripts/build.gradle (Module: app)` :

```js
dependencies {
    // ...
    implementation 'com.google.android.gms:play-services-location:21.0.1'
}
```

Synchronisez Gradle (un message doit vous le proposer en haut de la page).

#### a) Demander la permission

Avant, les permissions demandée par les applications étaient automatiquement données. Ce n'est plus le cas.

Il faudra donc mettre en place un code pour [demander la permission](https://developer.android.com/training/location/permissions?hl=fr).

Dans la fonction `onCreate`, à la fin du code déjà existant, ajoutez le code suivant :

```java
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        // ...

        // Demande la permission d'accès à la position
        ActivityResultLauncher<String[]> locationPermissionRequest =
                registerForActivityResult(new ActivityResultContracts
                                .RequestMultiplePermissions(), result -> {
                            Boolean fineLocationGranted = result.getOrDefault(
                                    android.Manifest.permission.ACCESS_FINE_LOCATION, false);
                            Boolean coarseLocationGranted = result.getOrDefault(
                                    android.Manifest.permission.ACCESS_COARSE_LOCATION,false);
                            if (fineLocationGranted != null && fineLocationGranted) {
                                // Precise location access granted.
                                Log.i("ENSG", "fineLocationGranted");
                            } else if (coarseLocationGranted != null && coarseLocationGranted) {
                                // Only approximate location access granted.
                                Log.i("ENSG", "coarseLocationGranted");
                            } else {
                                // No location access granted.
                                Log.e("ENSG", "noLocationGranted");
                            }
                        }
                );
        locationPermissionRequest.launch(new String[] {
                android.Manifest.permission.ACCESS_FINE_LOCATION,
                android.Manifest.permission.ACCESS_COARSE_LOCATION
        });
    }
```

Une fois que l'autorisation sera obtenue, nous pouvons lancer la géolocalisation.

#### b) Modification du code

Nous allons uniquement implémenter la seconde méthode.

Il faudra ajoutez un `FusedLocationProviderClient` en attribut de classe. C'est lui qui nous permet d'accéder à l'API :

```java
public class MapsActivity extends FragmentActivity implements OnMapReadyCallback {

    private GoogleMap mMap;
    private FusedLocationProviderClient mFusedLocationClient;

    // ...
}
```

Ensuite, dans la suite de la fonction `onCreate`, il va falloir instancier ce client :

```java
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        // ...

        // Instanciation du client de position
        mFusedLocationClient = LocationServices.getFusedLocationProviderClient(this);
    }
```

Maintenant, nous allons afficher un 2nd marqueur « Current position » donnant la position actuelle de l'utilisateur.

Créez une nouvelle méthode `getCurrentLocation`.

Nous allons dans un premier temps instancier une `LocationRequest`, qui va nous permettre de savoir avec quelle précision nous souhaitons suivre la position de l'utilisateur :

```java
    protected void getCurrentLocation() throws SecurityException {
        // Définition des paramètres de notre requête de mise à jour de la position
        LocationRequest locationRequest = new LocationRequest.Builder(Priority.PRIORITY_HIGH_ACCURACY, 5000)
                .setWaitForAccurateLocation(false)
                .setMinUpdateIntervalMillis(2000)
                .setMaxUpdateDelayMillis(10000)
                .build();
    }
```

Ensuite, nous allons créer un **callback** et exécuter la requête de position en le lui associant :

```java
    protected void getCurrentLocation() throws SecurityException {
        // ...

        // Callback de mise à jour
        LocationCallback locationCallback = new LocationCallback() {
            @Override
            public void onLocationResult(LocationResult locationResult) {
                for (Location location : locationResult.getLocations()) {
                    Log.i("ENSG", "currentLocation : " + location);
                }
            };
        };
        // Lancement de la demande
        mFusedLocationClient.requestLocationUpdates(locationRequest, locationCallback, null);
    }
```

Comme vous pouvez le voir, le second paramètre de la fonction est un callback, donc une classe avec une fonction à exécuter lorsque la position est trouvée. Vous pouvez simplement le déclarer juste avant.

Modifiez le code de `onLocationResult` pour afficher et déplacer le marqueur de la position courante (créez par exemple une fonction `public void setCurrentLocationMarkerLocation(Location location)`).

Voici un exemple du code attendu :

```java
public class MapsActivity extends FragmentActivity implements OnMapReadyCallback {

    private GoogleMap mMap;
    private FusedLocationProviderClient mFusedLocationClient;
    public Marker currentLocationMarker;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);

        ActivityMapsBinding binding = ActivityMapsBinding.inflate(getLayoutInflater());
        setContentView(binding.getRoot());

        // Obtain the SupportMapFragment and get notified when the map is ready to be used.
        SupportMapFragment mapFragment = (SupportMapFragment) getSupportFragmentManager()
                .findFragmentById(R.id.map);
        mapFragment.getMapAsync(this);

        // Demande la permission d'accès à la position
        ActivityResultLauncher<String[]> locationPermissionRequest =
                registerForActivityResult(new ActivityResultContracts
                                .RequestMultiplePermissions(), result -> {
                            Boolean fineLocationGranted = result.getOrDefault(
                                    android.Manifest.permission.ACCESS_FINE_LOCATION, false);
                            Boolean coarseLocationGranted = result.getOrDefault(
                                    android.Manifest.permission.ACCESS_COARSE_LOCATION,false);
                            if (fineLocationGranted != null && fineLocationGranted) {
                                // Precise location access granted.
                                Log.i("ENSG", "fineLocationGranted");
                                getLastLocation();
                                getCurrentLocation();
                            } else if (coarseLocationGranted != null && coarseLocationGranted) {
                                // Only approximate location access granted.
                                Log.i("ENSG", "coarseLocationGranted");
                                getLastLocation();
                                getCurrentLocation();
                            } else {
                                // No location access granted.
                                Log.e("ENSG", "noLocationGranted");
                            }
                        }
                );
        locationPermissionRequest.launch(new String[] {
                android.Manifest.permission.ACCESS_FINE_LOCATION,
                android.Manifest.permission.ACCESS_COARSE_LOCATION
        });

        // Instanciation du client de position
        mFusedLocationClient = LocationServices.getFusedLocationProviderClient(this);
    }

    protected void getCurrentLocation() throws SecurityException {
        // Définition des paramètres de notre requête de mise à jour de la position
        LocationRequest locationRequest = new LocationRequest.Builder(Priority.PRIORITY_HIGH_ACCURACY, 5000)
                .setWaitForAccurateLocation(false)
                .setMinUpdateIntervalMillis(2000)
                .setMaxUpdateDelayMillis(10000)
                .build();
        // Callback de mise à jour
        LocationCallback locationCallback = new LocationCallback() {
            @Override
            public void onLocationResult(LocationResult locationResult) {
                for (Location location : locationResult.getLocations()) {
                    Log.i("ENSG", "currentLocation : " + location);
                    setCurrentLocationMarkerLocation(location);
                }
            };
        };
        // Lancement de la demande
        mFusedLocationClient.requestLocationUpdates(locationRequest, locationCallback, null);
    }

    public void setCurrentLocationMarkerLocation(Location location) {
        // Est-ce que la carte est ok ?
        if (mMap != null) {
            LatLng position = new LatLng(location.getLatitude(), location.getLongitude());
            // Faut-il créer le marker ou juste le déplacer ?
            if (currentLocationMarker == null) {
                // Création
                currentLocationMarker = mMap.addMarker(new MarkerOptions().position(position).title("Current location"));
            } else {
                // Déplacement
                currentLocationMarker.setPosition(position);
            }
        } else {
            Log.w("ENSG", "Carte non initialisée. Impossible de mettre le currentLocation marker à " + location);
        }
    }

    @Override
    public void onMapReady(GoogleMap googleMap) {
        mMap = googleMap;

        // Add a marker in Sydney and move the camera
        LatLng ensg = new LatLng(48.8410837, 2.5875354);
        mMap.addMarker(new MarkerOptions().position(ensg).title("ENSG"));
        mMap.moveCamera(CameraUpdateFactory.newLatLngZoom(ensg, 14));

        // Affichage outils de zoom
        mMap.getUiSettings().setZoomControlsEnabled(true);
    }
}
```


### 5) Reverse Geocoding

Pour effectuer le géo-codage, nous allons utiliser la classe [Geocoder](https://developer.android.com/reference/android/location/Geocoder) fournie par l'API.

Notes :
* il y a de nouveaux strings à ajouter dans les ressources, pensez bien à les ajouter... (quand il y a la fonction `getString(R.string.identifiant)`) ;
* il vous faudra créer une fonction `setCurrentLocationMarkerTitle` qui, comme son nom l'indique, mettra à jour l'adresse de la position courante.

#### a) Ajout du géocodeur

Ajoutez un attribut de classe de type `Geocoder` et instanciez le dans la méthode `onCreate` :

```java
public class MapsActivity extends Fragment {

    // Autres attributs de classe...
    private Geocoder geocoder;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        // Début de la fonction...

        // Instanciation du géocodeur
        geocoder = new Geocoder(this, new Locale("fr"));
    }

    // Reste de la classe...
}
```

#### b) Appel au géocodeur

On crée une fonction pour lancer l'appel au service de géocodage :

```java
    public void geocode(Location location) {
        try {
            // Appel au géocodeur
            List<Address> addresses = geocoder.getFromLocation(location.getLatitude(), location.getLongitude(), 1);
            // Si une adresse est trouvée
            if (!addresses.isEmpty()) {
                Address address = addresses.get(0);
                // On va concaténer ses différentes parties
                StringBuilder bldr = new StringBuilder();
                for (int i = address.getMaxAddressLineIndex(); i >= 0; i--) {
                    String part = address.getAddressLine(i);
                    if (part != null && !part.equals("")) {
                        if (bldr.length() > 0) {
                            bldr.append(", ");
                        }
                        bldr.append(part);
                    }
                }
                // Et appeler la fonction permettant de modifier le titre du marqueur
                setCurrentLocationMarkerTitle(bldr.toString());
            } else {
                // TODO : Ajouter un message dans les ressources pour quand l'adresse n'est pas trouvée
                setCurrentLocationMarkerTitle(getString(R.string.no_address_found));
            }
        } catch (IOException e) {
            Log.e("ENSG", e.toString());
            // TODO : Ajouter un message dans les ressources pour quand il y a une erreur
            setCurrentLocationMarkerTitle(getString(R.string.unknown_error));
        }
    }
```

Fonction à appeler par exemple au moment de la mise à jours de la position du marqueur.

Il faut enfin faire une dernière fonction pour mettre à jour le titre du marker currentLocation et afficher un Toast : `setCurrentLocationMarkerTitle()`.

Voici le résultat :

![Géo-codage](screens/3_result_2.png)

## Les points à retenir

Il faut générer une clef pour utiliser l'API Google Maps.

Les Google Services sont un moyen simple de se géolocaliser :
* ou en utilisant la dernière position connue ;
* ou en s'abonnant aux mises à jour de position.


## Améliorations nécessaires

Dans le cas d'une vraie application, il est important d'arrêter l'abonnement aux mises à jours de position quand l'utilisateur ferme l'application et de les réactiver à l'ouverture (notamment pour économiser la batterie...).

On utilise pour cela les fonctions `onResume` et `onPause` :

```java

    @Override
    protected void onResume() {
        super.onResume();
        startLocationUpdates();
    }

    @Override
    protected void onPause() {
        super.onPause();
        stopLocationUpdates();
    }

    private void stopLocationUpdates() {
        if (mFusedLocationClient != null && locationCallback != null) {
            mFusedLocationClient.removeLocationUpdates(locationCallback);
        }
    }

```

Elle sont automatiquement exécutées à l'ouverture et à la fermeture de l'activité.

## Projet complet

Vous pouvez retrouver le projet complet ici : [https://gitlab.com/vsasyan/AndroidENSG-codes/-/tree/master/PositionName](https://gitlab.com/vsasyan/AndroidENSG-codes/-/tree/master/PositionName)

Aller au tutoriel suivant : [Photos](../4_photo/README.md)
