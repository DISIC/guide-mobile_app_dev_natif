# Guide de développement d'applications accessibles mobiles avec l'API Android

## Sommaire

  * [Sommaire](#sommaire)
  * [À qui s'adresse ce guide&nbsp;?](#%C3%A0-qui-sadresse-ce-guide&nbsp)
  * [Généralités sur Android et l'accessibilité](#g%C3%A9n%C3%A9ralit%C3%A9s-sur-android-et-laccessibilit%C3%A9)
  * [Utliser les éléments de base fournis par l'API Android](#utliser-les-%C3%A9l%C3%A9ments-de-base-fournis-par-lapi-android)
    * [Widgets compatibles](#widgets-compatibles)
    * [Widgets incompatibles](#widgets-incompatibles)
  * [Tester l'accessibilité](#tester-laccessibilit%C3%A9)
  * [Décrire les éléments d'interface](#d%C3%A9crire-les-%C3%A9l%C3%A9ments-dinterface)
    * [Exemples de création statique](#exemples-de-cr%C3%A9ation-statique)
      * [ImageView et ImageButton](#imageview-et-imagebutton)
      * [CheckBox](#checkbox)
      * [EditText](#edittext)
    * [Mise à jour dynamique de la description des contenus](#mise-%C3%A0-jour-dynamique-de-la-description-des-contenus)
  * [Permettre la modification de la taille des caractères](#permettre-la-modification-de-la-taille-des-caract%C3%A8res)
  * [Gérer le focus](#g%C3%A9rer-le-focus)
    * [Activer la prise de focus](#activer-la-prise-de-focus)
    * [Contrôler l'ordre de lecture](#contr%C3%B4ler-lordre-de-lecture)
  * [Afficher la prise de focus](#afficher-la-prise-de-focus)
  * [Masquer les éléments décoratifs](#masquer-les-%C3%A9l%C3%A9ments-d%C3%A9coratifs)
  * [Forcer la restitution des éléments importants non focusables](#forcer-la-restitution-des-%C3%A9l%C3%A9ments-importants-non-focusables)
  * [Regrouper des éléments](#regrouper-des-%C3%A9l%C3%A9ments)
  * [Gérer des événements liés à l'accessibilité](#g%C3%A9rer-des-%C3%A9v%C3%A9nements-li%C3%A9s-%C3%A0-laccessibilit%C3%A9)
    * [AccessibilityEvent](#accessibilityevent)
    * [Cheminement des événements](#cheminement-des-%C3%A9v%C3%A9nements)
    * [Récupérer plus d'informations sur le contexte avec AccessibilityNodeInfo](#r%C3%A9cup%C3%A9rer-plus-dinformations-sur-le-contexte-avec-accessibilitynodeinfo)
  * [Créer des vues personnalisées accessibles](#cr%C3%A9er-des-vues-personnalis%C3%A9es-accessibles)
    * [Gérer la navigation par contrôleurs directionnels](#g%C3%A9rer-la-navigation-par-contr%C3%B4leurs-directionnels)
    * [Implémenter les méthodes pour l'accessibilité](#impl%C3%A9menter-les-m%C3%A9thodes-pour-laccessibilit%C3%A9)
      * [onInitializeAccessibilityEvent()](#oninitializeaccessibilityevent)
      * [onInitializeAccessibilityNodeInfo()](#oninitializeaccessibilitynodeinfo)
      * [onPopulateAccessibilityEvent()](#onpopulateaccessibilityevent)
    * [Gérer le cas des gestes personnalisés](#g%C3%A9rer-le-cas-des-gestes-personnalis%C3%A9s)
  * [Pour aller plus loin](#pour-aller-plus-loin)
  * [Guides connexes](#guides-connexes)
  * [Ressources externes et références](#ressources-externes-et-r%C3%A9f%C3%A9rences)
  * [Licence](#licence)

## À qui s'adresse ce guide&nbsp;?

Ce guide présente des éléments de l'API d'Android utiles pour développer des applications accessibles aux personnes handicapées. Il s'adresse&nbsp;:

* Aux développeurs
* Aux concepteurs en charge de la rédaction de spécifications techniques
* Aux chefs de projets

Pré-requis&nbsp;:

* Maîtriser les langages Java et XML
* Maîtriser les principes de programmation sous Android
* Connaître les classes de base de l'environnement Android (notamment "activity" et "service")
* Maîtriser la gestion d'événements
* Maîtriser les concepts utilisés pour gérer les interfaces utilisateurs sous Android (vues, layouts, etc.) et les principaux éléments de l'API (TextView, CheckBox, RadioButton, etc.)


## Généralités sur Android et l'accessibilité

L'API Android met à disposition des développeurs des fonctionnalités permettant de créer des applications accessibles. Ces fonctionnalités sont présentes dans les classes qui permettent de créer les éléments de base d'une interface utilisateur ainsi que dans des classes dédiées à l'accessibilité.

Les applications Android peuvent être utilisées par des personnes en situation de handicap (visuel, moteur, etc.), grâce à l'activation de services dédiés à l'accessibilité sur l'appareil utilisé pour exécuter l'application et parfois grâce à l'utilisation de périphériques externes (claviers, commutateurs, plages braille, etc.). Les services d'accessibilité peuvent être activés en se rendant dans les paramètres de l'appareil, dans l'onglet "Accessibilité".

Google développe plusieurs services&nbsp;:

* Le lecteur d'écran TalkBack, qui vocalise la navigation à l'intérieur des applications. Il propose plusieurs fonctionnalités&nbsp;:
  * Exploration tactile&nbsp;: l'utilisateur explore le contenu de l'application en parcourant l'écran avec les doigts. TalkBack pilote un logiciel de synthèse vocale qui prononce les informations relatives aux éléments explorés positionnés sous les doigts de l'utilisateur. Dans ce mode, TalBack redéfinit les gestes "standards"&nbsp;: l'activation d'un élément se fait par exemple par un "double tap".
  * Navigation gestuelle simplifiée&nbsp;: TalkBack redéfinit des gestes permettant à l'utilisateur de naviguer facilement entre les différents éléments de l'interface.
  * Retour haptique&nbsp;: lorsque des éléments d'interface sont activés (entrée dans une zone de saisie par exemple), TalkBack le signale à l'utilisateur par des vibrations.
* BrailleBack, qui permet aux utilisateur possédant des appareils de lecture braille éphémère de les connecter via USB ou Bluetooth à leur appareil Android pour lire et interagir avec le contenu des applications.

De même que certains constructeurs proposent des versions modifiées du système Android, il existe des versions modifiées des services d'accessibilité&nbsp;: c'est par exemple le cas des certains téléphones de la marque Samsung qui sont livrés avec le service Galaxy TalkBack, une version du lecteur d'écran basée sur celui développé par Google.

La simple activation de services d'accessibilité ne suffit pas à rendre une application pleinement accessible&nbsp;: le développeur doit mettre en oeuvre des techniques particulières lors du codage de l'application pour rendre son contenu exploitable par les services d'accessibilité.

Lorsque l'interface utilisateur d'une application est constituée d'éléments de base fournis par le <span lang="en">framework</span> Android, la mise en oeuvre de l'accessibilité est relativement simple. La démarche est la suivante&nbsp;:

1. Décrire les éléments d'interface pour que leur contenu soit restitué par les services d'accessibilité
2. Faire en sorte que tous les éléments qui appellent une interaction puissent être atteints par un contrôleur directionnel (trackball, D-pad, etc.)
3. Faire en sorte que les messages audio soient accompagnés par des notifications visuelles et/ou haptiques, pour être perçues par les personnes sourdes ou malentendantes

Afin de développer des interfaces plus complexes et personnalisées qui nécessitent d'étendre la classe View, il sera nécessaire de s'assurer que ces éléments sont compatibles avec les services d'accessibilité en redéfinissant certaines méthodes, notamment pour gérer correctement les événements liés à l'accessibilité.

## Utliser les éléments de base fournis par l'API Android

L'API d'Android fournit des éléments de base (<span lang="en">widgets,
views</span>) qui implémentent correctement l'accessibilité dans la plupart
des cas. Il convient en premier lieu de privilégier l'utilisation de ces
éléments et de n'en définir de nouveaux qu'en cas de nécessité.

Nous listons ci-dessous des éléments compatibles et non compatibles avec les
services d'accessibilité. Ils ont été testés avec les versions d'Android 4.3,
4.4 et 5.1.

### Widgets compatibles

Les <span lang="en">widgets</span> suivants ont été testés et peuvent être utilisés pour créer des interfaces compatibles avec les services d'accessibilité (en veillant à décrire les contenus et à gérer correctement le focus comme indiqué dans les sections précédentes).

* EditText
* CheckBox
* ToggleButton
* Switch
* RadiGroup et RadioButton
* Chronometer
* TectClock
* AutoCompleteTextView
* Button
* ProgressBar
* RatingBar
* SeekBar
* NumberPicker
* TextSwitcher


<span lang="en">Layouts</span>&nbsp;:

* LinearLayout
* TableLayout

Vues&nbsp;:

* TextView
* ImageView
* ListView
* SearchView
* WebView

### Widgets incompatibles
En revanche, les <span lang="en">widgets</span> suivants ne sont pas compatibles avec les services d'accessibilité. Ils devront être adaptés au cas par cas&nbsp;:

* TabHost&nbsp;: le contenu des onglets n'est pas restitué correctement par les services d'accessibilité
* DatePicker
* TimePicker
* Spinner
* ToolBar

Vues&nbsp;:

* CalendarView
* ExpandableListView
* StackView

## Tester l'accessibilité

Dans l'API Android, les fonctionnalités relatives à l'accessibilité sont "stabilisées" depuis la version 4.0 (il n'y pas de changement majeur dans l'API). Cependant, les services d'accessibilité (TalkBack notamment) évoluent rapidement et changent parfois l'interprétation qu'ils font de certains éléments d'interface ou de certains événements liés à l'accessibilité. Par exemple, la version 4.3.1 de TalkBack parue en octobre 2015 est en mesure de restituer les changements de valeurs des <span lang="en">"sliders"</span>, ce qui n'était pas le cas dans les versions précédents (on devait alors utiliser l'attribut `contentDescription` pour en restituer le contenu). L'interprétation par les services d'accessibilité des <span lang="en">widgets</span> courants est relativement stable&nbsp;: les tests réalisés n'ont pas permis de mettre en évidence des régressions. En revanche, l'interprétation de vues personnalisées peut varier, mais ces variations sont extrêmement complexes à documenter. Par ailleurs, certains constructeurs de matériel peuvent ajouter des surcouches au système Android, ce qui a parfois un impact sur l'accessibilité. Il est ainsi nécessaire de tester l'accessibilité d'une application Android afin de vérifier que les recommandations présentées ci-dessous sont bien prises en compte par les services d'accessibilité (voir le [Guide d'audit d'applications mobiles](https://github.com/DISIC/guide-mobile_app_audit)).

## Décrire les éléments d'interface
Un grand nombre d'éléments d'une interface utilisateur fournissent des informations sur leur usage ou sur leur signification grâce à des indications visuelles. Par exemple, une application de prise de note utilise un élément `ImageButton` contenant l'image d'un signe "plus"  pour indiquer que l'utilisateur peut ajouter une nouvelle note. Un composant `EditText` peut disposer d'une étiquette qui permet de préciser l'information que l'utilisateur doit saisir. Un utilisateur aveugle ne peut pas percevoir ces indications visuelles. Il est donc nécessaire d'employer un moyen pour ajouter une alternative à ces indications. Pour cela il faut utiliser l'attribut `android:contentDescription` dans la description XML d'un élément d'interface ou la méthode `setContentDescription` dans le code Java. Le texte qui se trouve dans cette description ne sera pas affiché sur l'écran mais sera uniquement traité par les services d'accessibilité lorsque l'utilisateur les aura activés. Avec TalkBack, le texte sera prononcé par le synthétiseur vocal&nbsp;; avec BrailleBack, le texte apparaîtra à proximité de l'élément sur l'afficheur braille.

### Exemples de création statique

#### ImageView et ImageButton
`ImageView` permet d'afficher une image à l'écran. `ImageButton` est une sous-classe d'`ImageView` qui lui ajoute les comportements d'un bouton "standard". Ces classes possèdent un attribut `android:src` qui désigne une ressource graphique correspondant à l'image qui sera affichée et utilisée pour représenter le bouton dans le cas d'`ImageButton`. Cette ressource graphique ne pouvant pas être interprétée par les services d'accessibilité, il convient d'utiliser l'attribut `android:contentDescription` pour fournir une alternative textuelle à cette image.

Exemple avec `ImageButton`&nbsp;:
```xml
  <ImageButton
    android:id="@+id/imageButton2"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:src="@drawable/ic_launcher"
    android:contentDescription="Ajouter une note">
```

Comportement attendu avec TalkBack&nbsp;: annonce du contenu de l'attribut `android:contentDescription` précédé du mot "bouton" lors de la prise de focus ou du survol de l'image


#### CheckBox
`CheckBox` permet d'implémenter une case à cocher.

```xml
  <CheckBox android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:id="@+id/checkBox"
    android:text="Re-commander"
    contentDescription="Re-commander des pivoines"/>
```

L'attribut `android:text` contient le texte affiché à l'écran. Il peut être pertinent d'ajouter des éléments de contexte dans l'attribut `android:contentDescription` pour fournir à l'utilisateur des précisions sur l'action qui sera réalisée lors de l'activation ou la désactivation de la case. Par exemple, une vue peut présenter plusieurs cases à cocher disposées en regard d'une liste de produits dont l'utilisateur doit indiquer si oui ou non il souhaite recommander un/des élément(s). Si seulement l'attribut "android:text" est renseigné avec le texte "Recommander", l'utilisateur ne disposera pas de suffisamment d'informations pour savoir quelle action réalisera le fait de cocher la case. Ajouter "Recommander des pivoines" dans l'attribut `android:contentDescription` lui permettra de connaître plus précisément l'action réalisée lorsqu'il utilise un service d'accessibilité.

Comportement attendu avec TalkBack&nbsp;: prononciation de l'état de la case, suivi de "Case à cocher" puis du contenu de la description.

#### EditText
`EditText` permet de créer une zone de saisie de texte. Pour `EditText`, contrairement à la plupart des autres <span lang="en">widgets</span> disponibles dans le <span lang="en">framework</span> Android, l'attribut `android:contentDescription` ne peut pas être utilisé pour véhiculer une information décrivant l'élément (le contenu de l'attribut ne sera pas restitué par les services d'accessibilité).

L'utilisation de `android:hint` permet à l'utilisateur de comprendre la nature de l'information attendue&nbsp;: le contenu de `android:hint` est une aide à la saisie affichée lorsque la zone de saisie est vide, et lorsque la zone est complétée, le lecteur d'écran restitue le texte saisi au lieu de l'attribut.
```xml
  <EditText android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:id="@+id/editText"
    android:hint="Adresse email" />
```

Cependant, le contenu de `android:hint` disparaît dès que l'utilisateur saisit un caractère et ne peut plus être restitué, même si la zone est vidée&nbsp;: cela créé des difficultés notamment dans lorsque plusieurs champs de saisie sont proposés dans une même vue. Le meilleur moyen d'assurer l'accessibilité d'une zone de saisie est d''y associer une étiquette visible en utilisant l'attribut `android:labelFor`.

En XML&nbsp;:
```xml
  <TextView
    android:labelFor="Code postal"
    .../>

  <EditText android:id="@+id/edit_text"/>
```

En Java, les fonctions `setLabelFor(View label)` ou `setLabeledBy(View label)` peuvent être utilisées.

Comportement attendu avec TalkBack&nbsp;:

* Le contenu de `android:hint` est restitué uniquement lorsqu'aucun texte n'a été saisi&nbsp;; lorsqu'un texte a été saisi, celui-ci est restitué par TalkBack&nbsp;; lorsqu'un texte a été saisi puis supprimé, TalkBack indique seulement la présence de la zone de saisie sans restituer le contenu de `android:hint`.
* Le contenu de `android:labelFor` est restitué quel que soit le contexte.

### Mise à jour dynamique de la description des contenus
Lorsque le contenu d'un élément évolue au cours du temps, il est nécessaire de veiller à mettre à jour sa description, notamment lorsque l'état de l'élément ne peut être interprété par un autre moyen que cette description. Pour cela, il faut utiliser la méthode `setContentDescription` dans le code Java de l'application. La mise à jour dynamique de la description est par exemple utile dans les cas suivants, lors de la création de <span lang="en">widgets</span> personnalisés&nbsp;:

* Pour sélectionner une couleur dans une palette&nbsp;: les couleurs doivent être identifiées par une description textuelle et la description indiquant la couleur sélectionnée doit être mise à jour afin que l'utilisateur en ait connaissance lorsqu'il reprendra le focus sur le <span lang="en">widget</span>
* Pour sélectionner une date&nbsp;: la description peut être utilisée pour rappeler l'intégralité de la date sélectionnée après la mise à jour d'un des éléments (jour, mois, année) par l'utilisateur
* Pour indiquer le changement de fonction d'un bouton image lorsque sa signification est véhiculée par la forme ou la couleur&nbsp: un bouton "Lecture" devenant "Pause" par exemple

Une méthode pour mettre à jour la description est d'utiliser des <span lang="en">listeners</span> pour surveiller le changement d'état d'un <span lang="en">widget</span> afin de prendre les mesures nécessaires au moment approprié.

L'exemple ci-dessous met en oeuvre une `SeekBar`, un <span lang="en">widget</span> qui permet de saisir une valeur prise dans un intervalle déterminé. TalkBack, dans ses version inférieures à 4.3.1, ne surveillait pas le changement d'état de la `SeekBar` et n'était donc pas en mesure d'informer l'utilisateur des changements de valeurs. Ce comportement est identique avec BrailleBack. Le code proposé surveille la modification de la `SeekBar` et met à jour dynamiquement la description fournie aux services d'accessibilité lorsque la valeur change. Ainsi, la valeur sera indiquée à l'utilisateur après chaque changement, ce qui n'aurait pas été le cas par défaut.
NB&nbsp;: ce code est donné à titre d'exemple&nbsp; avec TalkBack 4.3.1, son utilisation provoque une double restitution lors du changement de la valeur de la `SeekBar`. Il doit donc être considéré comme une illustration du principe de mise à jour dynamique de la description, mais son utilisation n'est pas nécessairement pertinente dans tous les contextes.

Description XML&nbsp;:
```xml
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
	      android:layout_width="fill_parent"
	      android:layout_height="fill_parent"
	      android:orientation="vertical">

  <SeekBar android:layout_width="fill_parent"
	   android:layout_height="wrap_content"
	   android:id="@+id/reading_speed"
	   android:max="10"/>
</LinearLayout>
```

Code Java&nbsp;:
```java
  package com.braillenet.android_accessibility;

  import android.app.Activity;
  import android.os.Bundle;
  import android.widget.SeekBar;

  public class ProgressBarActivity extends Activity {

    private SeekBar mReadingSpeed;
    String name;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
      super.onCreate(savedInstanceState);
      setContentView(R.layout.progressbar);
      mReadingSpeed = (SeekBar)findViewById(R.id.reading_speed);
      name = "Vitesse de lecture";
      mReadingSpeed.setOnSeekBarChangeListener(new SeekBarListener(name));
      mReadingSpeed.setContentDescription(name);
    }

    private class SeekBarListener implements SeekBar.OnSeekBarChangeListener {
      private String mName;
      public SeekBarListener(String name) {
        mName = name;
      }
      public void onProgressChanged(SeekBar seekBar, int progress,
        boolean fromUser) {
        int percent = 100 * seekBar.getProgress() / seekBar.getMax();
        seekBar.setContentDescription(mName + " " + percent + "%");
      }
      public void onStartTrackingTouch(SeekBar seekBar) {
      }
      public void onStopTrackingTouch(SeekBar seekBar) {
        seekBar.setContentDescription(mName);
      }
    }
  }
```

Comportement attendu avec TalkBack&nbsp: lorsque la valeur est modifiée (on peut utiliser les bouton "+" et "-" du volume pour cela), TalkBack annonce le message "Vitesse de lecture x%".

## Permettre la modification de la taille des caractères

Android propose à l'utilisateur de paramétrer la taille des caractères (menu Paramètres, Accessibilité puis Taille des caractères&nbsp;: Petit, Normal ou Grand). Pour qu'une application réponde correctement à ce paramétrage, il est nécessaire d'utiliser l'unité "sp" (<span lang="en">scale-independent pixel</span>) lors de la définition des tailles de texte.
Exemple&nbsp;: `<TextView ... android:textSize="12sp"/>`

Il est possible de connaître le facteur d'ajustement sélectionné par l'utilisateur appliqué à la taille des caractères, ce qui permet d'adapter le comportement de l'application en conséquence (modifier le nombre de colonnes d'un tableau par exemple). Pour cela, il faut accéder à la valeur `Settings.System.FONT_SCALE` de la table `Settings.System.CONTENT_URI`. (1.0f est la valeur correspondant à la taille "normale").

Exemple&nbsp;:
```java
  ContentResolver r = getContentResolver();
  Cursor c = r.query(Settings.System.CONTENT_URI,
                     new String[] { Settings.System.VALUE },
		     Settings.System.NAME + "= ?",
		     new String[] { Settings.System.FONT_SCALE }, null);
  float fs = c.getFloat(0);
  if(fs ...) {
     ..
  } else {
    ...
  }
```

## Gérer le focus

Il est nécessaire de permettre aux utilisateurs de naviguer entre les différents éléments de l'interface grâce à un contrôleur directionnel. Un tel contrôleur peut être matériel (trackball, D-Pad, flèches d'un clavier, etc.) ou virtuel comme le clavier fourni par une application ou le mode de navigation gestuelle disponible depuis Android 4.1. La navigation grâce aux contrôleurs directionnels est largement utilisée, notamment par les personnes en situation de handicap moteur.

Pour s'assurer qu'une telle navigation fonctionne, il est nécessaire de vérifier que tous les éléments d'une application soient atteignables sans utiliser les interactions tactiles. Il est par ailleurs nécessaire de vérifier que cliquer sur le bouton central (ou le bouton OK) d'un contrôleur directionnel a le même effet que de toucher un élément sur l'écran possédant le focus.

### Activer la prise de focus

Un élément d'une interface utilisateur est atteignable en utilisant un contrôleur directionnel lorsque l'attribut `android:focusable` vaut "true"&nbsp;: cela permet à l'utilisateur de prendre le focus sur cet élément et d'interagir avec le contenu. Les éléments de base fournis par le <span lang="en">framework</span> Android sont focusables par défaut. Le fait qu'un élément soit porteur ou non du focus est signalé par une modification de son apparence.
L'API d'Android fournit plusieurs méthodes permettant de contrôler qu'un élément est bien focusable, ainsi que de donner le focus à un élément&nbsp;:

* `setFocusable()`&nbsp;: indique que le focus pourra être pris sur l'élément
* `isFocusable()`&nbsp;: indique si un élément peut recevoir le focus
* `requestFocus()`&nbsp;: permet de donner le focus à un élément

Si une vue ne peut pas recevoir le focus par défaut, il est possible d'utiliser l'attribut `android:focusable` avec la valeur "true" ou d'utiliser la méthode `setFocusable()` pour changer ce comportement.

### Contrôler l'ordre de lecture

Lorsqu'un utilisateur navigue dans une application grâce à un contrôleur directionnel, le focus passe d'un élément de l'interface à l'autre selon un ordre déterminé par un algorithme qui cherche l'élément le plus proche dans la direction souhaitée. Dans certains cas, l'ordre calculé par cet algorithme peut ne pas être cohérent avec la logique de l'application. Pour contourner ce problème, il est possible de définir des relations entre les différents éléments. Les attributs `android:nextFocusDown` (respectivement `android:nextFocusUp`, `android:nextFocusLeft` et `android:nextFocusRight`) permettent de définir le prochain élément qui recevra le focus lorsque l'utilisateur se déplacera vers le bas (respectivement vers le haut, à gauche et à droite).

Exemple&nbsp;:
```xml
  <LinearLayout android:orientation="horizontal"
        ... >
    <EditText android:id="@+id/edit"
      android:nextFocusDown=”@+id/text”
      ... />
    <TextView android:id="@+id/text"
      android:focusable=”true”
      android:text="Je suis une TextView focusable"
      android:nextFocusUp=”@id/edit”
      ... />
  </LinearLayout>
```

Cette technique peut par exemple s'appliquer pour faire en sorte que le focus décrive un cercle autour d'un ensemble d'éléments constituant une horloge&nbsp;: à 12h, on souhaite passer à 1h en appuyant sur "droite" ou "bas" et à 11h en appuyant sur "gauche" ou "bas". On aura des relations du type&nbsp;:
```xml
  <Button
    android:text="12"
    android:id="@+id/button12h"
    android:nextFocusUp="@+id/button11h"
    android:nextFocusLeft="@+id/button11h"
    android:nextFocusRight="@+id/button1h"
    android:nextFocusDown="@+id/button1h"
  </Button>
  <Button
    android:text="1"
    android:id="@+id/button1h"
    android:layout_below="@+id/button12h"
    android:layout_toRightOf="@+id/button12h"
    android:nextFocusUp="@+id/button12h"
    android:nextFocusLeft="@+id/button12h"
    android:nextFocusRight="@+id/button2h"
    android:nextFocusDown="@+id/button2h">
  </Button>
  ...
  <Button
    android:text="11h"
    android:id="@+id/button11h"
    android:layout_below="@+id/button12h"
    android:layout_toLeftOf="@+id/button12h"
    android:nextFocusUp="@+id/button10h"
    android:nextFocusLeft="@+id/button10h"
    android:nextFocusRight="@+id/button12h"
    android:nextFocusDown="@+id/button12h">
  </Button>
```

## Afficher la prise de focus

Il est nécessaire d'indiquer visuellement quel élément possède le focus. Pour cela, il faut créer un `drawable` et lui affecter l'attribut `android:drawable` pour l'état `state_focusable= "true"`.

Exemple&nbsp;:
```xml
  <selector xmlns:android="http://schemas.android.com/apk/res/android">
    ...
    <item android:state_focused="true" android:state_enabled="true" android:drawable="@drawable/button_is_focused" />
  </selector>
```


## Masquer les éléments décoratifs

Il est nécessaire de masquer les éléments décoratifs (par exemple une image n'apportant pas d'information supplémentaire dans le contexte dans lequel elle apparaît) pour faire en sorte qu'ils ne soient pas restitués par TalkBack. Pour cela, il faut utiliser l'attribut `android:importantForAccessibility="no"`.

## Forcer la restitution des éléments importants non focusables

Certains éléments non focusables, comme les images non cliquables, ne sont par défaut pas restitués par TalkBack&nbsp;: ce comportement est problématique si ces éléments ne sont pas décoratifs (i.e. qu'ils contiennent des informations essentiels à la compréhension). Il convient alors de forcer leur restitution par les services d'accessibilité en utilisant l'attribut `android:importantForAccessibility="yes"`.

## Regrouper des éléments

Pour faciliter la restitution par les outils d'assistance, il peut être utile de grouper les informations contenues dans plusieurs éléments en une seule description afin de minimiser le nombre d'opérations que devra effectuer l'utilisateur pour y accéder. Par exemple, il est pertinent de regrouper l'intitulé d'un produit, son poids et son prix en un texte qui sera restitué à l'utilisateur en une seule fois.

Pour cela, il est nécessaire de créer un <span lang="en">layout</span> parent avec l'attribut `android:importantForAccessibility="yes"`, de lui associer des fils qui seront créés avec l'attribut `android:importantForAccessibility="no"`, et de placer la description de l'ensemble des éléments regroupés dans l'attribut `android:contentDescription` du <span lang="en">layout</span> parent.

## Gérer des événements liés à l'accessibilité

Dans les éléments de base du <span lang="en">framework</span> Android, lorsque du contenu est sélectionné, que le focus change ou qu'un élément est survolé, des événements du type `AccessibilityEvent` sont créés. Ces événements sont reçus et interprétés par les services d'accessibilité afin par exemple de fournir des fonctionnalités de restitution vocale aux utilisateurs.

Générer des événements peut être utile lors de la conception d'une interface utilisateur qui met en oeuvre des vues personnalisées et capturer des événement sert aux services d'accessibilité pour interpréter le comportement d'une application.

### AccessibilityEvent

Un événement peut être généré en utilisant la méthode `sendAccessibilityEvent(int)`, avec un paramètre représentant le type d'événement qui s'est produit. 
La liste complète des typées d'événements est disponible dans la description de la classe <a href="http://developer.android.com/reference/android/view/accessibility/AccessibilityEvent.html" lang="en">AccessibilityEvent</a>. Citons par exemple&nbsp;:

* `TYPE_VIEW_FOCUSED`&nbsp;: déclenché lorsque le focus est positionné sur une vue
* `TYPE_VIEW_CLICKED`&nbsp;: déclenché lorsqu'un clic est réalisé sur une vue (`Button, CompoundButton`, etc.)
* `TYPE_VIEW_TEXT_CHANGED`&nbsp;: déclenché lorsque le texte est modifié dans une zone de saisie (`EditText`)
* `TYPE_ANNOUNCEMENT`&nbsp;: événement "générique" qui peut être utilisé lorsqu'aucun autre type d'événement ne convient, pour indiquer un changement de contexte ou envoyer une information spécifiques destinée à être restituée par les services d'accessibilité. À noter que la classe `View` met à disposition la méthode `announceForAccessibility(CharSequence text)` qui est un moyen simple d'envoyer cet événement.

Chaque événement dispose de propriétés accessibles via des méthodes permettant de connaître des informations à son sujet lorsqu'il est intercepté par un service d'accessibilité. Par exemple&nbsp;:

* `getClassName()`&nbsp;: le nom de la classe d'où a été émis l'événement
* `getPackageName()`&nbsp;: le nom du package d'où a été émis l'événement
* `getContentDescription()`&nbsp;: le contenu de l'attribut `contentDescription` de la source
* `isPassword()`&nbsp;: si la source est un champ destiné à recevoir un mot de passe

D'autres méthodes permettent d'initialiser ou de modifier des propriétés de la source de l'événement. Par exemple&nbsp;:

* `setContentDescription`&nbsp;: définit la description (correspond à l'information contenue dans l'attribut `android:contentDescription` vue plus haut)
* `setPassword`&nbsp;: définit si un élément est destiné à recevoir un mot de passe

### Cheminement des événements

En plus de la méthode `sendAccessibilityEvent`, la classe `View` propose d'autres méthodes pour gérer les événements liés à l'accessibilité&nbsp;:

* `sendAccessibilityEventUnchecked`&nbsp;: cette méthode est utilisée lorsque le code appelant gère directement la vérification de l'activation des fonctions liées à l'accessibilité (en appelant `AccessibilityManager.isEnabled()`).
* `onPopulateAccessibilityEvent()`&nbsp;: définit le texte qui sera prononcé lors du déclenchement de l'`AccessibilityEvent` pour la vue concernée.
* `onInitializeAccessibilityEvent()`&nbsp;: méthode permettant d'obtenir des informations supplémentaires concernant l'état de la vue.
* `onInitializeAccessibilityNodeInfo()`&nbsp;: fournit des informations concernant l'état de la vue aux services d'accessibilité.
* `onRequestSendAccessibilityEvent()`&nbsp;: méthode appelée lorsqu'un enfant d'une vue a généré un `AccessibilityEvent`. Cela permet par exemple à la vue parente de modifier l'événement de l'enfant pour y ajouter des informations.

Pour comprendre le rôle des méthodes mentionnées ci-dessus, il est nécessaire de connaître le cheminement d'un événement&nbsp;:

* Lorsqu'un service d'accessibilité est activité et que l'utilisateur réalise une action (clic, prise de focus, survol, etc.), la vue est notifiée grâce à un appel à `sendAccessibilityEvent` (ou `sendAccessibilityEventUnchecked`)&nbsp;;
* Quand elles sont appelées, les méthodes `sendAccessibilityEvent()` et `sendAccessibilityEventUnchecked()` emploient un mécanisme de <span lang="en">callback</span> qui fait appel à `onInitializeAccessibilityEvent()` pour initialiser l'événement&nbsp;;
* Une fois initialisé, si le type de l'événement requiert qu'il soit propagé avec de l'information textuelle, la vue reçoit un appel à `dispatchPopulateAccessibilityEvent()`&nbsp;;
* Par un mécanisme de <span lang="en">callback</span>, `dispatchPopulateAccessibilityEvent()` appelle `onPopulateAccessibilityEvent()`&nbsp;;
* La vue fait ensuite remonter l'événement dans la hiérarchie des vues en appelant `requestSendAccessibilityEvent()` sur la vue "parent". Chaque vue "parent" a alors la possibilité d'enrichir les informations concernant l'accessibilité en ajoutant une `AccessibilityRecord`, jusqu'à ce que l'événement atteigne la vue racine&nbsp;;
* Une fois à la racine, l'événement est envoyé à l'`AccessibilityManager` par `sendAccessibilityEvent()`.

### Récupérer plus d'informations sur le contexte avec AccessibilityNodeInfo
Le rôle principal d'un événement est d'exposer suffisamment d'information à un service d'accessibilité pour que celui-ci fournisse lui-même des informations pertinentes à l'utilisateur. Parfois, un service d'accessibilité peut avoir besoin de davantage d'information que celle véhiculée par l'événement. Dans ce cas, il est possible de récupérer un objet de type `AccessibilityNodeInfo` qui représente l'état d'une vue, ce qui permet à un service d'accessibilité d'explorer le contenu de la fenêtre. (Pour des raisons de sécurité, l'accès à la source d'un événement est un privilège qui doit être explicitement demandé par une application.)

`AccessibilityNodeInfo` représente un noeud du contenu de la fenêtre ainsi que les actions qui peuvent être demandées depuis la source. De nombreuses méthodes permettent alors d'explorer le contenu de la fenêtre. Par exemple&nbsp;:

* `findAccessibilityNodeInfosByViewId`&nbsp;: renvoie une liste d'éléments de type `AccessibilityNodeInfo` correspondant à un identifiant
* `findFocus`&nbsp;: trouve la vue qui possède le focus
(Se référer à la classe  <a href="http://developer.android.com/reference/android/view/accessibility/AccessibilityNodeInfo.html" lang="en">AccessibilityNodeInfo</a> pour une liste exhaustive.)

## Créer des vues personnalisées accessibles
Le <span lang="en">framework</span> Android offre un nombre importants de composants de base&nbsp;: des <span lang="en">widgets</span> (`Button, CheckBox`, etc.) et des <span lang="en">layouts</span> (`LinearLayout, FrameLayout`, etc.). Il est toutefois possible de créer des composants personnalisés, en étendant la classe `View`&nbsp;: cela permet par exemple de contrôler précisément l'apparence et les fonctionnalités d'un élément. Il est alors nécessaire de s'assurer que ces éléments personnalisés sont accessibles.

### Gérer la navigation par contrôleurs directionnels

Sur la plupart des appareils, cliquer dans une vue en utilisant un contrôleur directionnel envoie un `KeyEvent` de type `KEYCODE_DPAD_CENTER` à la vue qui possède le focus. Toutes les vues Android de base traitent de manière appropriée ce type `KEYCODE_DPAD_CENTER`. Lorsqu'une vue personnalisée est mise en oeuvre, il faut veiller à ce que cet événement ait le même effet que toucher la vue sur l'écran tactile. Par ailleurs, il est aussi nécessaire de traiter l'événement `KEYCODE_ENTER` de la même façon que `KEYCODE_DPAD_CENTER`, ceci afin de faciliter la navigation au clavier.

Par exemple&nbsp;:
```java
  public boolean onKeyUp(int keyCode, KeyEvent event) {
    if ((keyCode == KeyEvent.KEYCODE_DPAD_CENTER) || (keyCode == KeyEvent.KEYCODE_ENTER)) {
      ...
    }
  }
```

### Implémenter les méthodes pour l'accessibilité

Afin de supporter l'accessibilité dans une vue personnalisée, il est nécessaire de surcharger et d'implémenter les méthodes décrites à la fin de la section "Gérer des événements liés à l'accessibilité".

Avant de donner des exemples d'implémentation de ces méthodes pour créer des vues personnalisées accessibles, il est important de comprendre l'ordre dans lequel sont gérés les événements liés à l'accessibilité.

#### onInitializeAccessibilityEvent()

Cette méthode est appelée par le système pour obtenir de l'information concernant l'état d'une vue. Si un composant personnalisé propose des fonctionnalités supplémentaires par rapport à un composant dont il hérite, il convient de surcharger la méthode `onInitializeAccessibilityEvent` et d'ajouter les informations supplémentaires liées au composant personnalisé. Pour surcharger la méthode, il faut d'abord appeler la méthode de la classe héritée et ensuite modifier des propriétés qui n'ont pas été initialisées par la méthode de la classe héritée.

Supposons par exemple que l'on souhaite étendre la classe `BaseToggleButton` pour faire en sorte que le bouton proposé soit coché par défaut. On aura un code du type&nbsp;:
```java
  public static class AccessibleCompoundButtonInheritance extends BaseToggleButton {
    ...
    // Cette méthode sera appelée à chaque appel de sendAccessibilityEvent()
    @Override
    public void onInitializeAccessibilityEvent(AccessibilityEvent event) {
      // On appelle l'implémentation de onInitializeAccessibilityEvent de la
      // classe héritée puis on initialise la propriété "checked" qui n'est
      // pas présente dans la classe héritée
      super.onInitializeAccessibilityEvent(event);
      event.setChecked(isChecked());
    }
  }
```


#### onInitializeAccessibilityNodeInfo()

`onInitializeAccessibilityNodeInfo` permet d'initialiser des informations concernant l'état de la vue qui serviront aux services d'accessibilité lorsqu'ils consulteront la source (type `AccessibilityNodeInfo`) d'un événement. Comme précédemment, cette méthode doit être surchargée pour fournir des informations complémentaires en plus de celles fournies par la classe héritée.

Les objets de type `AccessibilityNodeInfo` créés par cette méthode sont utilisés par les services d'accessibilité pour explorer la hiérarchie d'une vue qui a généré un événement d'accessibilité après que celui-ci ait été reçu, ce qui permet aux services d'accessibilité d'obtenir plus d'éléments sur le contexte d'apparition de l'événement.

Une implémentation de `onInitializeAccessibilityNodeInfo` est par exemple&nbsp;:
```java
  public static class AccessibleCompoundButtonInheritance extends BaseToggleButton {
    ...
    @Override
    public void onInitializeAccessibilityNodeInfo(AccessibilityNodeInfo info) {
      super.onInitializeAccessibilityNodeInfo(info);
      // On appelle l'implémentation de la classe héritée et on ajoute
      // des propriétés: checkable, isChecked, et le texte
      info.setCheckable(true);
      info.setChecked(isChecked());
      CharSequence text = getText();
      if (!TextUtils.isEmpty(text)) {
        info.setText(text);
      }
    }
  }
```


#### onPopulateAccessibilityEvent()
Chaque événement `AccessibilityEvent` dispose d'un ensemble de propriétés qui décrivent l'état courant de la vue. Lors de la création d'une vue personnalisée, il est nécessaire de fournir des informations sur le contexte et sur les caractéristiques de la vue&nbsp;: ce peut être l'étiquette d'un bouton, mais aussi des informations supplémentaires qui seront fournies à l'utilisateur.

La méthode `onPopulateAccessibilityEvent()` permet de fournir ces informations supplémentaires sur l'état d'une vue&nbsp,: elle est appelée automatiquement par le système pour chaque `AccessibilityEvent`. Cette méthode est donc l'endroit approprié pour décrire l'état d'une vue personnalisée afin notamment de fournir des instructions spécifiques à l'utilisateur de services d'accessibilité.

Le code suivant permet par exemple de faire prononcer le message "faites glisser" par les services d'accessibilité lorsqu'ils prennent le focus sur la vue&nbsp;:
```java
  public class TestView extends View {
    ...

    // Méthode appelée pour propager les événements liés à l'accessibilité.
    public void onPopulateAccessibilityEvent(AccessibilityEvent event) {
      // On commence par appeler la méthode de la classe héritée
      super.onPopulateAccessibilityEvent(event);
      // On récupérère le type d'événement
      int eventType = event.getEventType();
      // S'il correspond à une prise de focus par un service d'accesibilité...
      if (eventType == AccessibilityEvent.TYPE_VIEW_ACCESSIBILITY_FOCUSED) {
        // ... on modifie l'événement pour ajotuer du texte qui sera
	// prononcé par le service
        event.getText().add("Faites glisser");
      }
    }
  }
```

### Gérer le cas des gestes personnalisés
Des vues personnalisées peuvent mettre en oeuvre des interactions gestuelles non standards. Ceci est réalisé en utilisant la méthode `onTouchEvent` pour détecter certains types d'événements. Les services d'accessibilité redéfinissent souvent les gestes de base qui permettent de naviguer à l'intérieur d'une application et les actions qui y sont associés. Il est donc nécessaire de s'assurer que les interactions gestuelles personnalisées resteront compatibles avec les services d'accessibilité, qu'elles permettront de générer des actions qui pourront être interprétées par eux et que ces actions pourront être réalisées par d'autres moyens que l'utilisation de l'écran tactile.

Le code qui génère la capture des événements tactiles doit réaliser les deux actions suivantes&nbsp;:

1. Générer un `AccessibilityEvent` approprié à l'action réalisée par clic
2. Permettre aux services d'accessibilité de réaliser l'action par un autre moyen que l'écran tactile

Pour cela, il est nécessaire de surcharger la méthode `performClick()` qui doit d'abord exécuter la méthode de la classe héritée puis exécuter les actions correspondant à l'événement "clic" capturé. La méthode `performClick()` doit ensuite être appelée.

Par exemple&nbsp;:
```java
  public boolean onTouchEvent(MotionEvent event) {
    super.onTouchEvent(event);
    switch (event.getAction()) {
      case MotionEvent....:
        ...
	return true;
      case MotionEvent....:
        // On capture un événement intéressant qui doit causer un clic
	// On produit ce clic en appelant performClick() : cela permettra
	// aux services d'accessibilité d'en être avertis
	performClick();
        return true;
    }
    return false;
  }

  @Override
  public boolean performClick() {
    // On appelle l'implémentation de la classe héritée qui fait tout le
    // travail: générer un AccessibilityEvent et capturer les événements de
    // clic avec onClick()
    super.performClick();
    // On réalise ensuite les actions particulières à réaliser dans
    // la vue personnalisée
    blabla();
    return true;
  }
```

## Pour aller plus loin
Il arrive que des implémentations sous forme de vues personnalisées ne permettent pas aux services d'accessibilité de récupérer suffisamment d'éléments de contexte pour restituer l'interface de manière satisfaisante. C'est par exemple le cas d'un ensemble de vues pour lequel une action sur l'une des vues modifie le contenu d'une ou de plusieurs autres vues. Dans ce cas, un service d'accessibilité ne pourra pas être averti qu'une action sur une vue en modifie une autre, car la relation entre les vues n'est pas explicite. Pour contourner ce problème, il est possible de créer des "vues virtuelles" qui ne seront exposées qu'aux seuls services d'accessibilité. Ces vues virtuelles permettent notamment de mieux représenter les informations et le comportement de l'interface et de faire en sorte que les services d'accessibilité aient accès des informations en rapport avec la "logique" de l'application.
La mise en oeuvre d'une vue virtuelle est complexe&nbsp;: elle consiste notamment à étendre la classe `AccessibilityNodeProvider`. Pour plus de détails, nous renvoyons à <a href="https://code.google.com/p/android-source-browsing/source/browse/samples/ApiDemos/src/com/example/android/apis/accessibility/AccessibilityNodeProviderActivity.java?repo=platform--development" lang="en">cet exemple fourni par Google</a>.

Pour des usages très spécifiques, les services d'accessibilité "standards" tels que TalkBack peuvent ne pas donner entière satisfaction&nbsp;: le concepteur d'une application peut souhaiter que celle-ci ait pleinement la main sur la manière dont seront restitués les éléments de son interface. Par exemple, ce peut être le cas d'applications qui nécessitent de piloter plusieurs outils de synthèse vocale pour distinguer des contenus de natures différentes (ce comportement n'est pas possible avec TalkBack)&nbsp;: utiliser une voix "standard" pour lire le contenu des éléments de l'interface utilisateur de l'application, et une voix de meilleure qualité pour lire le contenu d'un document. l'API d'Android répond à ce besoin en donnant au développeur la possibilité de créer ses propres services d'accessibilité. Le principal rôle d'un tel service sera de traiter les événements d'accessibilité et de les interpréter. Un service possède un type défini qui détermine le retour qu'il fournit à l'utilisateur&nbsp;: `FEEDBACK_SPOKEN, FEEDBACK_BRAILLE, FEEDBACK_VISUAL`, etc. Un service peut capturer tous les événements ou les filtrer selon leur type (capturer uniquement les prises de focus par exemple). Enfin, il peut avoir une portée restreinte à un <span lang="en">package</span> ou un ensemble de <span lang="en">packages</span> pour se limiter à une application précise, ou bien avoir une portée "globale" pour traiter toutes les applications du système. Pour créer un service, il est nécessaire d'étendre la classe `AccessibilityService`. Cette opération est complexe et nous revoyons à <a href="http://code.google.com/p/eyes-free/source/browse/trunk/documentation/ClockBackTutorial/Step-By-Step/src/com/google/android/marvin/clockback/ClockBackService.java?r=576" lang="en">cet exemple d'implémentation de service</a> proposé par Google pour plus de détails.

## Guides connexes

Les guides suivants peuvent être consultés en complément&nbsp;:

* [Guide d'audit d'applications mobiles](https://github.com/DISIC/guide-mobile_app_audit)
* [Guide de conception d'applications mobiles accessibles](https://github.com/DISIC/guide-mobile_app_conception)
* [Guide de développement d'applications mobiles accessibles avec Ionic et OnsenUI](https://github.com/DISIC/guide-mobile_app_dev_hybride)


## Ressources externes et références

Sources&nbsp;:

* <a href="http://developer.android.com/reference/packages.html" lang="en">API Android</a>
* <a href="http://developer.android.com/guide/topics/ui/accessibility/apps.htm l" lang="en">Making Applications Accessible</a>
* <a href="https://code.google.com/p/android-source-browsing/source/browse/samples/ApiDemos/src/com/example/android/apis/accessibility/?repo=platform--development" lang="en">Exemples d'utilisation de l'API Android</a>
* <a href="http://www.bbc.co.uk/guidelines/futuremedia/accessibility/mobile" lang="en">BBC Mobile Accessibility Prototype</a>

## Licence
Ce document est la propriété du Secrétariat général à la modernisation de l'action publique français (SGMAP). Il est placé sous la [licence ouverte 1.0 ou ultérieure](http://wiki.data.gouv.fr/wiki/Licence_Ouverte_/_Open_Licence), équivalente à une licence <i lang="en">Creative Commons BY</i>. Pour indiquer la paternité, ajouter un lien vers la version originale du document disponible sur le [compte <span lang="en">Github</span> de la DInSIC](https://github.com/DISIC).
