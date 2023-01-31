# Photo

Android permet également d'accéder aux capteurs d'image de votre téléphone.

Il y a deux manières de prendre des photos :
* en utilisant l'API de base ([camera2](https://developer.android.com/training/camera2) ou [CameraX](https://developer.android.com/training/camerax?hl=fr)) ;
* en utilisant une autre application photo installée sur le téléphone.

Nous verrons la première manière en utilisant CameraX.

## Objectifs

Cette partie permet de voir :
* comment ajouter les dépendances à CameraX ;
* comment afficher une prévisualisation de la photo ;
* comment prendre une photo et l’enregistrer.

## Le principe de l'application

L'application sera composée d'un écran permettant de visualiser et prendre une photo.

## Mise en place

Créez un nouveau projet appelé `CameraX` avec une première activité de type **Empty Activity** nommée `MainActivity`.

## Pas à pas des modifications

### 1) Dépendances

Ouvrez le fichier `build.gradle (Module: app)` et ajoutez les dépendances :

```js
dependencies {
    def camerax_version = "1.1.1"
    implementation "androidx.camera:camera-core:${camerax_version}"
    implementation "androidx.camera:camera-camera2:${camerax_version}"
    implementation "androidx.camera:camera-lifecycle:${camerax_version}"
    implementation "androidx.camera:camera-video:${camerax_version}"

    implementation "androidx.camera:camera-view:${camerax_version}"
    implementation "androidx.camera:camera-extensions:${camerax_version}"

    // ...
}
```

Vérifiez que dans le bloc "android" se soit bien indiqué de compiler avec Java 8 :

```js
android {
    // ...
    compileOptions {
        sourceCompatibility JavaVersion.VERSION_1_8
        targetCompatibility JavaVersion.VERSION_1_8
    }
    // ...
}
```

A la fin du bloc "android" ajoutez la *buildFeature* `viewBinding` :

```js
android {
    // ...
    buildFeatures {
        viewBinding true
    }
}
```

Synchronisez Gradle.

### 2) Affichage

Nous allons créer une activité avec un composant PreviewView permettant d'afficher l'image captée par l'appareil et un bouton pour prendre la photo. Reprenez le code suivant (**créez les ressources manquantes !**) :

```xml
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MainActivity">

    <RelativeLayout
        android:layout_width="match_parent"
        android:layout_height="match_parent">

        <androidx.camera.view.PreviewView
            android:id="@+id/viewFinder"
            android:layout_width="match_parent"
            android:layout_height="match_parent" />

        <Button
            android:id="@+id/image_capture_button"
            android:layout_width="110dp"
            android:layout_height="110dp"
            android:layout_marginBottom="50dp"
            android:text="@string/take_photo"
            android:layout_centerHorizontal="true"
            android:layout_alignParentBottom="true" />

    </RelativeLayout>

</androidx.constraintlayout.widget.ConstraintLayout>
```

### 3) Autorisation

Pour prendre des photos, vous avez besoin de deux autorisations (à ajouter avant la balise `application`) :

```xml
    <uses-feature android:name="android.hardware.camera.any" />
    <uses-permission android:name="android.permission.CAMERA" />
```

Déclarer la permission ne suffit pas. Il faudra également ajouter un code pour activer cette permission. Dans la fonction `onCreate`, ajouter le code suivante (**créez les ressources manquantes !**) :

```java
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        // ...

        // Vérification de l'autorisation d'accès la camera
        if (checkSelfPermission(android.Manifest.permission.CAMERA) != PackageManager.PERMISSION_GRANTED) {
            ActivityResultLauncher<String> requestPermissionLauncher =
                    registerForActivityResult(new ActivityResultContracts.RequestPermission(), isGranted -> {
                        if (!isGranted) {
                            Toast.makeText(MainActivity.this, getString(R.string.permission_needed), Toast.LENGTH_LONG).show();
                            Log.i("ENSG", "permission denied");
                        } else {
                            startCamera();
                        }
                    });
            requestPermissionLauncher.launch(android.Manifest.permission.CAMERA);
        } else {
            startCamera();
        }
    }
```

Et créez la méthode `startCamera()` :

```java
    private void startCamera() {
        Log.i("ENSG", "startCamera");
    }
```

Essayez de lancer l'application. A ce stade elle doit demander l'autorisation et on doit voir le log de la fonction `startCamera`.


### 4) Prévisualisation

La prochaine étape consiste à créer un lien entre l'image capturée par l'appareil et le composant graphique de prévisualisation.

Ajouter l'attribut imageCapture comme attribut de classe :

```java
public class MainActivity extends AppCompatActivity {
    private ImageCapture imageCapture;

    // ...
}
```

Ajoutez le code suivant dans la fonction `startCamera()` :

```java
        ListenableFuture<ProcessCameraProvider> cameraProviderFuture  = ProcessCameraProvider.getInstance(this);
        cameraProviderFuture.addListener(new Runnable() {
            @Override
            public void run() {
                // Used to bind the lifecycle of cameras to the lifecycle owner
                ProcessCameraProvider cameraProvider = null;
                try {
                    cameraProvider = cameraProviderFuture.get();

                    // Prévisualisation
                    PreviewView previewView = findViewById(R.id.viewFinder);
                    Preview preview = new Preview.Builder().build();
                    preview.setSurfaceProvider(previewView.getSurfaceProvider());

                    // Récupération de l'image
                    imageCapture = new ImageCapture.Builder().build();

                    // On choisi l'appareil au dos TODO : laisser le choix à l'utilisateur !
                    CameraSelector cameraSelector = CameraSelector.DEFAULT_BACK_CAMERA;

                    // On coupe les liens déjà existants si jamais
                    cameraProvider.unbindAll();
                    // On crée le lien avec notre previsualisation
                    cameraProvider.bindToLifecycle(MainActivity.this, cameraSelector, preview, imageCapture);
                } catch (ExecutionException e) {
                    Log.e("ENSG", "ExecutionException failure", e);
                } catch (InterruptedException e) {
                    Log.e("ENSG", "InterruptedException failure", e);
                } catch (Exception e) {
                    Log.e("ENSG", "Use case binding failed", e);
                }
            }
        }, getMainExecutor());
```

A ce niveau testez pour votre application. Vous devez voir ce que l'appareil va prendre, mais sans pouvoir le prendre... :)


### 5) Prise de photo

La prise de photo se fera via l'objet `imageCapture`.

Avant de prendre la photo, nous devons récupérer en endroit ou enregistrer le fichier.

Deux choix s'offrent à nous :
* enregistrer la photo dans le répertoire privée de notre application. Nous n'avons pas besoin d'autorisation pour cela, mais aucune autre application ne pourra y accéder...
* enregistrer la photo en utilisant l'API [MediaStore](https://developer.android.com/training/data-storage/shared?hl=fr). Disponible depuis Android 10, cette l'API permet d'accéder simplement aux fichiers partagés (entre application) pour lire les existant ou en écrire de nouveaux. Il n'y a [pas besoin d'autorisation](https://developer.android.com/training/data-storage/shared/media?hl=fr) pour ajouter un fichier.

Pour pouvoir insérer un fichier, nous avons besoin de le décrire : donner son nom, donner son type (image, vidéo, audio, ...). Cela permet au système de le référencer.

NB : il faudrait gérer l'ancienne méthode de création de fichiers pour les utilisateurs avant Android 10 (et là nous aurions besoin de l'autorisation d'écriture pour écrire en dehors de la zone réservée). Nous n'allons pas faire cela ici.

Ajoutez la méthode `takePicture` à votre classe `MainActivity` :

```java
public class MainActivity extends AppCompatActivity {
    // ...

    private void takePhoto() {
        // On vérifie que imageCapture est instancié
        if (imageCapture != null) {
            // Récupération d'une entrée MediaStore nommée selon la date et l'heure
            // 1) On converti la date en nom de fichier
            SimpleDateFormat dateFormatter = new SimpleDateFormat("yyyy-MM-dd-HH-mm-ss-SSS", Locale.US);
            String name = String.format("%s.jpeg", dateFormatter.format(System.currentTimeMillis()));
            // 2) On demande la création d'une entrée MediaStore pour notre image
            ContentValues contentValues = new ContentValues();
            // 2-a) On donne son nom
            contentValues.put(MediaStore.MediaColumns.DISPLAY_NAME, name);
            // 2-a) On donne son type
            contentValues.put(MediaStore.MediaColumns.MIME_TYPE, "image/jpeg");
            // 2-c) Résolution de l'entrée à partir des info données
            ContentResolver contentResolver = getApplicationContext().getContentResolver();
            ImageCapture.OutputFileOptions outputFileOptions = new ImageCapture.OutputFileOptions
                    .Builder(contentResolver, MediaStore.Images.Media.EXTERNAL_CONTENT_URI, contentValues)
                    .build();

            // On utilise le imageCapture pour prendre la photo (encore un système de callback s/e)
            imageCapture.takePicture(outputFileOptions, ContextCompat.getMainExecutor(this), new ImageCapture.OnImageSavedCallback() {
                @Override
                public void onImageSaved(@NonNull ImageCapture.OutputFileResults outputFileResults) {
                    // La photo a été prise et enregistrée
                    String msg = String.format("Photo capture succeeded: %s", name);
                    Toast.makeText(MainActivity.this, msg, Toast.LENGTH_SHORT).show();
                    Log.d("ENSG", msg);
                }

                @Override
                public void onError(@NonNull ImageCaptureException exception) {
                    // Erreur
                    String msg = String.format("Photo capture failed: %s", exception.getMessage());
                    Log.e("ENSG", msg, exception);
                }
            });
        } else {
            Log.e("ENSG", "imageCapture is null");
        }
    }
}
```

**Ajoutez également un écouteur d'événement sur le bouton de prise de photo pour que cette fonction soit exécutée.**

Vous pouvez alors lancer votre application et prendre des photos !


## Les points à retenir

L'API CameraX permet d'utiliser l'appareil photo du téléphone (relativement) simplement.

Vous pouvez sauver des fichiers :
* dans une zone commune à tout le téléphone (via `MediaStore`) ;
* dans une zone privée réservée à votre application.

## Projet complet

Vous pouvez retrouver le code de prise de photo ici : [https://gitlab.com/vsasyan/CameraX](https://gitlab.com/vsasyan/CameraX)
