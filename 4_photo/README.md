# Photo

Android permet également de prendre des photos.

Il y a deux manières :
* en utilisant l'API de base ;
* en utilisant une autre application photo installer sur le téléphone.

Nous verrons la seconde manière qui est plus simple pour commencer.

## Objectifs

Cette partie permet de voir :
* comment créer un `Intent` pour prendre des photos ;
* comment est géré le stockage de fichiers en Android.

## Le principe de l'application

L'application sera composée d'un écran de bienvenue (`MainActivity`) et d'une carte (MapsActivity). On va se concentrer sur la prise de photo géolocalisée (qui se fera depuis la MainActivity), une application complète pourrait par exemple permettre de voir les photos qui ont été prises sur la carte.

## Mise en place

Créer un nouveau projet appelé `GeoPicture` avec une première activité de type *Basic Activity* nommée `MainActivity`.

Créer une autre activité de type *maps* nommée `MapsActivity`.

## Pas à pas des modifications

### 1) Autorisation

Pour prendre des photos, vous avez besoin de deux autorisations :

```xml
    <uses-feature android:name="android.hardware.camera" android:required="true" />
    <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />
```

La première permet la prise de photo et la seconde permet d'écrire dans la mémoire de l'appareil.

Notes :

1) Il y a également une autorisation pour la lecture (`android.permission.READ_EXTERNAL_STORAGE`) cependant, l'autorisation d'écriture (`android.permission.WRITE_EXTERNAL_STORAGE`) comprend implicitement cette autorisation.

2) Depuis la version 18 du SDK, **toutes les applications** ont automatiquement un droit d'écriture et de lecture **dans une zone mémoire accessible à l'application seule** (là où sont d'ailleurs sauvés les fichiers SQLite). Cette zone mémoire est automatiquement supprimée à la désinstallation de l’application.

### 2) Chemin d'accès

Nous verrons *trois* accès possibles :
* celui de la mémoire privée réservée à cette application ;
* celui de la mémoire publique accessible à toutes les applications (mais pas référencée pour l'utilisateur) ;
* celui de la mémoire publique accessible à toutes les applications (et référencée pour l'utilisateur).

Voici les fonctions à appeler (respectivement pour l'accès privé, l'accès public non référencé et l'accès public référencé) :
* [`File getFilesDir();`](https://developer.android.com/reference/android/content/Context#getFilesDir()) ;
* [`File getExternalFilesDir(String name);`](https://developer.android.com/reference/android/content/Context#getExternalFilesDir(java.lang.String)) ;
* [`File getExternalStoragePublicDirectory(String name);`](https://developer.android.com/reference/android/os/Environment#getExternalStoragePublicDirectory(java.lang.String)).

Les deux dernières fonctions prennent un argument de type `String` qui doit être :
* [DIRECTORY_PICTURES](https://developer.android.com/reference/android/os/Environment#DIRECTORY_PICTURES) ;
* [DIRECTORY_MUSIC](https://developer.android.com/reference/android/os/Environment#DIRECTORY_MUSIC) ;
* [etc](https://developer.android.com/reference/android/os/Environment)...

Cela permet d'obtenir directement un dossier adapté au type de fichier que l'on souhaite sauvegarder.

La différence entre la mémoire publique référencée et la non référencée est en fait assez simple : vous avez une application de type Galerie qui vous permet de voir les photos prises sur votre téléphone. Une photo sauvée par votre application dans la mémoire "publique référencée" sera directement visible dans la Galerie alors qu'une image sauvée dans la mémoire "publique non référencée" ne le sera pas par défaut.

### 3) Préparation de l'application

Ajoutez un bouton `b_take_picture` à l'application, ainsi que les attributs stockant les composants graphiques, leur instanciation et l'ajout des écouteurs d'événements (appel d'une fonction `startPictureIntent` au clic bouton).

### 4) `startPictureIntent`

Le moyen le plus simple de prendre une photo est d'utiliser le système d'`Intent` pour ouvrir l'application photo de l'utilisateur et prendre.

Qui dit `Intent`, dit codes et clés pour communiquer... Donc nous allons créer une classe `Constants` avec un attribut `static final int REQUEST_IMAGE_CAPTURE = 1;`.

Ensuite, nous allons mettre dans la fonction `startPictureIntent` le code qui permet de créer l'`Intent` :

```java
    private void startPictureIntent() {
        Intent takePictureIntent = new Intent(MediaStore.ACTION_IMAGE_CAPTURE);
        // Ensure that there's a camera activity to handle the intent
        if (takePictureIntent.resolveActivity(getPackageManager()) != null) {
           startActivityForResult(takePictureIntent, Constants.REQUEST_IMAGE_CAPTURE);
        }
    }
```

Nous utilisons la fonction `startActivityForResult`, si personne ne répond à notre appel, notre application va planter...
C'est pour cela que l'on utilise la fonction `resolveActivity` : elle permet de vérifier l’existence d'au moins une application pour prendre une photo.

### 5) `onActivityResult`

Pour récupérer le résultat de notre appel, il faut ajouter une fonction `protected void onActivityResult(int requestCode, int resultCode, Intent data)`.

La fonction permet de récupérer un aperçus de l'image prise :

```java
    @Override
    protected void onActivityResult(int requestCode, int resultCode, Intent data) {
        super.onActivityResult(requestCode, resultCode, data);
        if (requestCode == Constants.REQUEST_IMAGE_CAPTURE && resultCode == RESULT_OK) {
            // Get result
            Bundle extras = data.getExtras();
            Bitmap imageBitmap = (Bitmap) extras.get("data");
            // imageBitmap est un aperçu, la preuve :
            Toast.makeText(this, String.format(Locale.getDefault(), "%d x %d", imageBitmap.getWidth(), imageBitmap.getHeight()), Toast.LENGTH_SHORT).show();
            // Chez moi ça ne retourne qu'une image de 160 x 120...
        }
    }
```

La fonction récupère la donnée si le résultat est positif (l'utilisateur a pris une photo).

Comme vous pouvez le voir, l'`Intent` ne retourne qu'un aperçu... Pour effectivement sauvegarder l'image, il faut créer un fichier dont on passera l'adresse en extra à l'`Intent`.

(A ce stade, lancez l'application pour vérifier qu'elle affiche bien un `Toast` avec les dimensions de l'aperçu.)

### 6) L'image complète

Cette fonction va créer le fichier (il faut ajouter un `String mCurrentPhotoPath` en attribut de la classe `MainActivity`) :

```java
    private File createImageFile() throws IOException {
        // Create an image file name
        String timeStamp = new SimpleDateFormat("yyyyMMdd_HHmmss").format(new Date());
        String imageFileName = "JPEG_" + timeStamp + "_";
        File storageDir = getExternalStoragePublicDirectory(DIRECTORY_PICTURES);
        File image = File.createTempFile(
            imageFileName,  /* file */
            ".jpg",         /* suffix */
            storageDir      /* directory */
        );

        // Save a file: path for use with ACTION_VIEW intents
        mCurrentPhotoPath = image.getAbsolutePath();
        return image;
    }
```

Note : importez les packages :

```java
import java.text.SimpleDateFormat;
import java.util.Date;
```

Ensuite, nous allons modifier le code de la fonction `startPictureIntent` :

```java
    private void startPictureIntent() {
        Intent takePictureIntent = new Intent(MediaStore.ACTION_IMAGE_CAPTURE);
        // Ensure that there's a camera activity to handle the intent
        if (takePictureIntent.resolveActivity(getPackageManager()) != null) {
            // Create the File where the photo should go
            File photoFile = null;
            try {
                photoFile = createImageFile();
            } catch (IOException ex) {
                // Error occurred while creating the File
                Toast.makeText(this, "Error: impossible to create picture file.", Toast.LENGTH_SHORT).show();
            }
            if (photoFile != null) {
                // Continue only if the File was successfully created
                Uri photoURI = FileProvider.getUriForFile(this, getString(R.string.file_provider_authority), photoFile);
                takePictureIntent.putExtra(MediaStore.EXTRA_OUTPUT, photoURI);
                startActivityForResult(takePictureIntent, Constants.REQUEST_IMAGE_CAPTURE);
            } else {
                // Else be sure to erase the path
                mCurrentPhotoPath = null;
            }
        }
    }
```

Une ligne doit vous sembler mystérieuse :

```java
                Uri photoURI = FileProvider.getUriForFile(this, getString(R.string.file_provider_authority), photoFile);
```

Cette manière de faire permet d'autoriser l'application photo que l'on utilise pour prendre la photo à écrire le fichier. Si on a pris une zone de stockage privée réservée à notre application, Android va laisser l'application photo accéder au fichier que l'on précise. Alors qu'en temps normal, cette zone est privée.

Il faut configurer ce `FileProvider` en ajoutant au fichier `AndroidManifest.xml` :

```xml
<!-- ... -->

<application>
    
    <!-- ... -->
    
    <provider
        android:name="androidx.core.content.FileProvider"
        android:authorities="@string/file_provider_authority"
        android:exported="false"
        android:grantUriPermissions="true">
        <meta-data
            android:name="android.support.FILE_PROVIDER_PATHS"
            android:resource="@xml/file_paths"></meta-data>
    </provider>

</application>
```

Le deuxième argument de la fonction `getUriForFile` et la balise `android:authorities` doivent être égaux (j'utilise donc une ressource commune pour m'en assurer : `<string name="file_provider_authority">fr.ign.vsasyan.fileprovider</string>`).

Vous voyez également que le bout de code fait référence à une ressource `@xml/file_paths` (soit le fichier : `res/xml/file_paths.xml`) que l'on va ajouter :

```xml
<?xml version="1.0" encoding="utf-8"?>
<paths xmlns:android="http://schemas.android.com/apk/res/android">
    <files-path name="my_files_getFilesDir" path="." />
    <external-path name="my_images_getExternalFilesDir" path="Android/data/fr.ign.vsasyan.geopicture/files/Pictures" />
    <external-path name="my_images_getExternalStoragePublicDirectory" path="/Pictures" />
</paths>
```

Il faut créer un nouveau dossier de ressources :
* clic-droit sur le dossier `res` ;
* New => Android Resources Directory ;
* nommez-le `xml`.

Puis lui ajouter un fichier de ressources :
* clic-droit sur le dossier `xml` crée ;
* New => XML Resources File ;
* nommez-le `file_paths`.

**Ligne *4*, mettez bien à jour l'attribut `path` pour qu'il corresponde au package de votre application !**

Ce document donne la liste des répertoires où doit se trouver le fichier rendu accessible au moment de l'`Intent`. Le chemin change selon l'endroit où vous enregistrez le fichier (voir [2) Chemin d'accès](#2-chemin-daccès)), j'ai donc ajouté tous les chemins possibles si besoin.
Vous pouvez vous amuser à changer la fonction appelée (dans votre méthode `createImageFile`) et commenter la ligne associée (dans le fichier `file_paths.xml`), vous verrez alors que ça plante au niveau de `FileProvider.getUriForFile`.

Il faut maintenant modifier la fonction `onActivityResult` car il n'y aura plus d'`Intent` retourné :

```java
    @Override
    protected void onActivityResult(int requestCode, int resultCode, Intent data) {
        if (requestCode == Constants.REQUEST_IMAGE_CAPTURE && resultCode == RESULT_OK) {
            // On a data == null car ou on récupère une miniature, ou on enregistre la photo, mais on ne peut pas faire les deux...
            // Mais resultCode == RESULT_OK donc la photo est bien sauvegardée
            // Ajoutons l'image à la Galerie
            galleryAddPic();
        }
    }

    private void galleryAddPic() {
        Intent mediaScanIntent = new Intent(Intent.ACTION_MEDIA_SCANNER_SCAN_FILE);
        File f = new File(mCurrentPhotoPath);
        Uri contentUri = Uri.fromFile(f);
        mediaScanIntent.setData(contentUri);
        this.sendBroadcast(mediaScanIntent);
    }
```

### 7) Ajoutons un peu de Géomatique

Maintenant nous allons supposer que notre application localisait l'utilisateur en arrière plan ([Voir TD 3](../3_google_services/)) et qu'il y a deux attributs `Double lat, lng;` comme attributs de classe.

Nous pouvons modifier la fonction `onActivityResult` pour qu'elle ajoute la position GPS au fichier :

```java
    @Override
    protected void onActivityResult(int requestCode, int resultCode, Intent data) {
        if (requestCode == Constants.REQUEST_IMAGE_CAPTURE && resultCode == RESULT_OK) {
            // Ajoutons l'image à la Galerie
            galleryAddPic();
            // Ajoutons des info de position
            addPicExifInfo();
            // Sauvons en base !
            addPictureToDataBase();
        }
    }

    protected void addPicExifInfo() {
        if (mCurrentPhotoPath != null && lat != null && lng != null) {
            try {
                ExifInterface exif = new ExifInterface(mCurrentPhotoPath);
                exif.setAttribute(ExifInterface.TAG_GPS_LATITUDE, PositionConverter.getLatTagGPS(lat));
                exif.setAttribute(ExifInterface.TAG_GPS_LONGITUDE, PositionConverter.getLngTagGPS(lng));
                exif.setAttribute(ExifInterface.TAG_GPS_LATITUDE_REF, PositionConverter.getLatRefTagGps(lat));
                exif.setAttribute(ExifInterface.TAG_GPS_LONGITUDE_REF, PositionConverter.getLngRefTagGps(lng));
                exif.saveAttributes();
            } catch (Exception exception) {
                Toast.makeText(this, String.format("Error when trying to add EXIF attributes: %s", exception), Toast.LENGTH_SHORT).show();
            }
        }
    }

    protected void addPictureToDataBase() {
        if (mCurrentPhotoPath != null && lat != null && lng != null) {
            //ImageBitmap preview = MAKE_A_PREVIEW();
            //Picture picture = new Picture(lat, lng, mCurrentPhotoPath, preview);
            //pictureDAO.create(picture);
        }
    }
```

On ajoute les données de localisions au fichier principal et on sauvegarde les données de la photo et une miniature dans une base.

Voici une classe pour convertir la position au bon format pour les tags exif :

```java
final class PositionConverter {

    static String getLatTagGPS(Double lat) {
        double latitude = Math.abs(lat);
        int num1Lat = (int)Math.floor(latitude);
        int num2Lat = (int)Math.floor((latitude - num1Lat) * 60);
        double num3Lat = (latitude - ((double)num1Lat+((double)num2Lat/60))) * 3600000;
        return num1Lat+"/1,"+num2Lat+"/1,"+num3Lat+"/1000";
    }

    static String getLngTagGPS(Double lng) {
        double longitude = Math.abs(lng);
        int num1Lon = (int)Math.floor(longitude);
        int num2Lon = (int)Math.floor((longitude - num1Lon) * 60);
        double num3Lon = (longitude - ((double)num1Lon+((double)num2Lon/60))) * 3600000;
        return num1Lon+"/1,"+num2Lon+"/1,"+num3Lon+"/1000";
    }

    static String getLatRefTagGps(Double lat) {
        if (lat > 0) {return "N";} else {return "S";}
    }

    static String getLngRefTagGps(Double lng) {
        if (lng > 0) {return "E";} else {return "W";}
    }
}
```

## Les points à retenir

Les `Intent` permettent également de démarrer d'autres applications.

Vous pouvez sauver des fichiers :
* dans une zone commune à tout le téléphone ;
* dans une zone privée réservée à votre application.

## Projet complet

Vous pouvez retrouver le code de prise de photo ici : [https://gitlab.com/vsasyan/GeoPicture](https://gitlab.com/vsasyan/GeoPicture)
