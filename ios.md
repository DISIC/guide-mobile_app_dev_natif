# Guide de développement d'applications mobiles accessibles avec l'API iOS

## Sommaire


  * [Sommaire](#sommaire)
  * [À qui s'adresse ce guide&nbsp;?](#%C3%A0-qui-sadresse-ce-guide&nbsp)
  * [ Généralités sur iOS et l'accessibilité](#g%C3%A9n%C3%A9ralit%C3%A9s-sur-ios-et-laccessibilit%C3%A9)
    * [ L'accessibilité dans iOS](#laccessibilit%C3%A9-dans-ios)
    * [L'API UI Accessibility](#lapi-ui-accessibility)
    * [Les attributs pour l'accessibilité](#les-attributs-pour-laccessibilit%C3%A9)
  * [Utiliser les éléments de base fournis par UIKIT](#utiliser-les-%C3%A9l%C3%A9ments-de-base-fournis-par-uikit)
    * [Widgets compatibles](#widgets-compatibles)
    * [Widgets incompatibles](#widgets-incompatibles)
  * [Tester l'accessibilité](#tester-laccessibilit%C3%A9)
  * [Déclarer l'accessibilité des vues](#d%C3%A9clarer-laccessibilit%C3%A9-des-vues)
    * [Les vues individuelles](#les-vues-individuelles)
    * [Les vues "container"](#les-vues-container)
  * [Décrire les éléments d'interface](#d%C3%A9crire-les-%C3%A9l%C3%A9ments-dinterface)
  * [ Identifier les caractéristiques des éléments d'interface](#identifier-les-caract%C3%A9ristiques-des-%C3%A9l%C3%A9ments-dinterface)
  * [Définir la zone de restitution d'un élément](#d%C3%A9finir-la-zone-de-restitution-dun-%C3%A9l%C3%A9ment)
  * [Mettre à jour les descriptions et caractéristiques](#mettre-%C3%A0-jour-les-descriptions-et-caract%C3%A9ristiques)
  * [Obtenir des informations sur l'état des options d'accessibilité](#obtenir-des-informations-sur-l%C3%A9tat-des-options-daccessibilit%C3%A9)
  * [Envoyer des notifications aux outils d'assistance](#envoyer-des-notifications-aux-outils-dassistance)
  * [Capturer et traiter les notifications envoyées par UIKit](#capturer-et-traiter-les-notifications-envoy%C3%A9es-par-uikit)
  * [Quelques cas particuliers](#quelques-cas-particuliers)
    * [Les modales](#les-modales)
    * [Les vues tableau (TableView)](#les-vues-tableau-tableview)
  * [Guides connexes](#guides-connexes)
  * [Ressources externes et références](#ressources-externes-et-r%C3%A9f%C3%A9rences)
  * [Licence](#licence)

## À qui s'adresse ce guide&nbsp;?

Ce guide présente des éléments de l'API d'iOS utiles pour développer des applications accessibles aux personnes handicapées. Il s'adresse&nbsp;:

* Aux développeurs
* Aux concepteurs en charge de la rédaction de spécifications techniques
* Aux chefs de projets

Pré-requis&nbsp;:

* Maîtriser le langage Objective C
* Maîtriser les principes de programmation sous iOS
* Maîtriser les concepts utilisés pour gérer les interfaces utilisateurs sous iOS
* Savoir utiliser XCode

## Généralités sur iOS et l'accessibilité

### L'accessibilité dans iOS
L'API iOS met à disposition des développeurs des fonctionnalités permettant de créer des applications accessibles. Ces fonctionnalités sont présentes dans les classes qui permettent de créer les éléments de base d'une interface utilisateur ainsi que dans des classes dédiées à l'accessibilité.

Les applications iOS peuvent être utilisées par des personnes en situation de handicap (visuel, moteur, etc.), grâce à l'activation de fonctionnalités dédiées à l'accessibilité sur l'appareil utilisé pour exécuter les applications et parfois grâce à l'utilisation de périphériques externes (claviers, commutateurs, plages braille, etc.). Les fonctionnalités intégrées directement dans iOS sont par exemple&nbsp;:

* Zoom&nbsp;: possibilité d'agrandir le contenu de l'écran
* Affichage en blanc sur fond noir (inversion des couleurs)
* Sortie audio en mono seulement&nbsp;: mixe les deux canaux d'un signal sonore en un seul
* Auto-complétion&nbsp;: prononce les suggestions de corrections pendant la saisie d'un texte par l'utilisateur
* Contrôle vocal&nbsp;: permet à un utilisateur d'accéder à certaines fonctionnalités de l'appareil en parlant
* Lecteur d'écran&nbsp;: iOS est livré avec le logiciel VoiceOver qui permet à un utilisateur déficient visuel de contrôler son appareil sans voir l'écran (prononciation du contenu vocalement, redéfinition des gestes, pilotage d'un afficheur braille, etc.)

Les fonctionnalités d'accessibilité peuvent être activées en se rendant dans le menu "Réglages" de l'appareil, puis dans l'onglet "Accessibilité".

La simple activation de fonctionnalités d'accessibilité ne suffit pas à rendre une application pleinement accessible&nbsp;: le développeur doit mettre en oeuvre des techniques particulières lors du codage de l'application pour rendre son contenu exploitable par les logiciels prenant en charge l'accessibilité.

Dans iOS, une application est dite accessible lorsque tous les éléments de son interface avec lesquels l'utilisateur peut interagir sont accessibles, c'est-à-dire lorsque chaque élément se déclare comme un élément porteur d'accessibilité. Par ailleurs, un élément doit porter des informations précises et pertinentes qui pourront être interprétées par les logiciels prenant en charge l'accessibilité&nbsp;: position sur l'écran, nom, comportement, valeur et type.

### L'API UI Accessibility

Depuis la version 3.0, iOS dispose d'une API dédiée à l'accessibilité ("UI Accessibility"). Cette API permet de fournir aux logiciels prenant en charge l'accessibilité toutes les informations utiles pour décrire l'interface utilisateur d'une application.

L'API UI Accessibility fait partie de UIKit. Ses fonctionnalités sont implémentés par défaut dans les vues et éléments d'interfaces de base d'iOS. Ainsi, des applications accessibles pourront facilement être créées tans que ces éléments de bases seront utilisées. Si des interfaces graphiques plus complexes doivent être créées (par l'utilisation de vues personnalisées), les travaux pour les rendre accessibles seront plus importants.

L'API UI Accessibility est constituée de deux protocoles, d'une classe, d'une fonction et d'un ensemble de constantes&nbsp;:

* Le protocole `UIAccessibility`&nbsp;: les objets qui implémentent ce protocole portent des informations relatives à leur accessibilité (à savoir s'ils déclarent être accessibles ou non) et fournissent des informations les décrivant. Les vues et éléments de base implémentent ce protocole par défaut.
* Le protocole `UIAccessibilityContainer`&nbsp;: ce protocole permet à une sous-classe d'UIView de rendre tout ou partie des objets qu'elle contient accessibles en tant qu'éléments distincts. Ceci sert lorsque les objets contenus dans une vue ne sont pas eux-mêmes des sous-classes d'UIView qui ne sont ainsi pas automatiquement considérés comme accessibles.
* La classe `UIAccessibilityElement`&nbsp;: cette classe définit un objet qui peut être renvoyé via le protocole UIAccessibilityContainer. Une instance de UIAccessibilityContainer peut être créée pour représenter un élément qui n'est pas automatiquement considéré comme étant accessible (par exemple, un objet qui n'hérite pas d'UIView).
* Des constantes (définies dans `UIAccessibilityConstants.h`)&nbsp;: ce sont les caractéristiques exposées par un élément et les notifications qu'une application peut envoyer.

### Les attributs pour l'accessibilité

Les contrôles et vues contiennent des attributs permettant de fournir des informations concernant leur accessibilité aux applications d'assistance. Ce sont par exemple&nbsp;:

* `isAccessibilityElement`&nbsp;: indique si l'élément est accessible.
* `accessibilityLabel`&nbsp;: donne une courte description d'un élément qui permet de comprendre ce qu'il contient ou ce qu'il permet de faire. Cet description ne doit pas comporter d'information concernant le type d'élément. Exemple&nbsp;: "Play", "Ajouter", etc. Le contenu de cet attribut sera lu par VoiceOver à la place du titre du composant s'il en possède un&nbsp;: dans ce cas, il permet d'expliciter ou de préciser le titre du composant.
* `accessibilityLanguage`&nbsp;: indique la langue utilisée pour décrire l'élément. Cette information est utiles pour les outils de synthèse vocale afin que la voix appropriée soit automatiquement sélectionnée.
* `accessibilityTraits`&nbsp;: une contient une combinaison d'une ou plusieurs caractéristiques, qui décrit l'état, le comportement ou le rôle d'un élément.
* `accessibilityHint`&nbsp;: indique une brève description qui permet à l'utilisateur de savoir ce que produit une action sur un élément.
* `accessibilityValue`&nbsp;: la valeur actuelle d'un élément. Par exemple, 30% pour un potentiomètre permettant de choisir le volume du son.
* `accessibilityFrame`&nbsp;: la position d'un élément sur l'écran.
* `accessibilityElementIsHidden`&nbsp;: indique si un élément doit être restitué ou non par VoiceOver.


Les éléments porteurs d'accessibilité contiennent ces attributs&nbsp;: ces informations peuvent être fournies dans l'<span lang="en">Interface Builder</span> au moment de la création de l'interface ou par le programmeur dans le code Objective C. Les attributs `accessibilityLabel` et `accessibilityFrame` sont toujours requis. L'attribut `accessibilityValue` est utile lorsque le contenu d'un élément est modifiable et qu'il ne peut pas être décrit par `accessibilityLabel`.

## Utiliser les éléments de base fournis par UIKIT

UIKit fournit des éléments de base (<span lang="en">widgets</span>) qui implémentent correctement l'accessibilité. Il convient en premier lieu de privilégier l'utilisation de ces éléments et de n'en définir de nouveaux qu'en cas de nécessité. Nous listons ci-dessous des éléments compatibles et non compatibles avec les outils d'accessibilité. Ils ont été testés avec les versions d'iOS 6, 7, 8 et 9.


### Widgets compatibles

Vues&nbsp;:

* UIAlertView
* UIPickerView
* UIScrollView
* UITableView
* UIWebView

Les composants&nbsp;:

* UIButton
* UIImage
* UILabel
* UISegmentedControl
* UISlider
* UISwitch
* UITextField

### Widgets incompatibles
En revanche, les <span lang="en">widgets</span> suivants ne sont pas directement compatibles avec les outils d'accessibilité. Ils devront être adaptés au cas par acas&nbsp;:

* CollectionView
* UIPageControl
* MapKitView

## Tester l'accessibilité

Dans l'API iOS, les fonctionnalités relatives à l'accessibilité sont "stabilisées" depuis la version 5 (il n'y pas de changement majeur dans l'API). Cependant, les outils d'accessibilité (VoiceOver notamment) évoluent rapidement et changent parfois l'interprétation qu'ils font de certains éléments d'interface ou de certaines notifications liées à l'accessibilité. L'interprétation par les services d'accessibilité des <span lang="en">widgets</span> courants est relativement stable&nbsp;: les tests réalisés n'ont pas permis de mettre en évidence des régressions. En revanche, l'interprétation de vues personnalisées peut varier, mais ces variations sont extrêmement complexes à documenter. Il est ainsi nécessaire de tester l'accessibilité d'une application iOS afin de vérifier que les recommandations présentées ci-dessous sont bien prises en compte par les services d'accessibilité (voir le [Guide d'audit d'applications mobiles](https://github.com/DISIC/guide-mobile_app_audit)).

## Déclarer l'accessibilité des vues

C'est au développeur de s'assurer de l'accessibilité des vues personnalisées qu'il crée. Il existe deux situations&nbsp;: les vues individuelles et les vues "containers".

### Les vues individuelles
Ce sont des vues qui contiennent un unique élément avec lequel l'utilisateur peut interagir. Il est nécessaire de déclarer ces vues comme étant accessibles. Plusieurs moyens sont à disposition&nbsp;: soit dans l'<span lang="en">Interface Builder</span>, soit dans le code Objective C de l'application.

Une première possibilité est de déclarer l'attribut `setIsAccessibilityElement` lors de l'instanciation de la vue&nbsp;:
```
  @implementation VuePersController
  - (id)init
   {
     _view = [[[VuePers alloc] initWithFrame:CGRectZero] autorelease];
     [_view setIsAccessibilityElement:YES];
     ...
   }
```

Une autre option est d'implémenter la méthode `isAccessibilityElement` du protocole `UIAccessibility`&nbsp;:
```
  @implementation VuePers
  - (BOOL)isAccessibilityElement
   {
      return YES;
   }
```

### Les vues "container"

Les vues "container" regroupent d'autres vues qui constituent les éléments de l'interface avec lesquels l'utilisateur interagit. Ce sont ces éléments regroupés qui doivent être déclarés comme étant accessibles&nbsp;: la vue "container" ne doit pas être déclarée comme étant accessible, mais doit déclarer la liste des éléments accessibles qu'elle contient. Pour cela, il faut utiliser le protocole `UIAccessibilityContainer`.

Déclarer la liste des éléments accessibles contenus dans la vue en utilisant `accessibleElements`&nbsp;:
```
  - (NSArray *)accessibleElements
  {
     if ( _accessibleElements != nil )
     {
        return _accessibleElements;
     }
     _accessibleElements = [[NSMutableArray alloc] init];
     /* Crée un premier élément et l'initialise comme étant contenu dans la vue */
     UIAccessibilityElement *element1 = [[[UIAccessibilityElement alloc] initWithAccessibilityContainer:self] autorelease];
     /* Ajoute des attributs à cet élément */
     [_accessibleElements addObject:element1];
     /* Crée un deuxième élément... */
     UIAccessibilityElement *element2 = [[[UIAccessibilityElement alloc] initWithAccessibilityContainer:self] autorelease];
     /* Lui ajoute des attributs */
     [_accessibleElements addObject:element2];
     /* etc... */
     ...
     return _accessibleElements;
  }
```

Il faut ensuite déclarer que cette vue n'est pas accessible&nbsp;:
```
  - (BOOL)isAccessibilityElement
  {
    return NO;
  }
```

Enfin, implémenter les méthodes `accessibilityElementCount`, `accessibilityElementAtIndex` et `indexOfAccessibilityElement` du protocole `UIAccessibilityContainer`&nbsp;:

```
  - (NSInteger)accessibilityElementCount
  {
     return [[self accessibleElements] count];
  }

  - (id)accessibilityElementAtIndex:(NSInteger)index
  {
     return [[self accessibleElements] objectAtIndex:index];
  }

  - (NSInteger)indexOfAccessibilityElement:(id)element
  {
     return [[self accessibleElements] indexOfObject:element];
  }
```

## Décrire les éléments d'interface

Un grand nombre d'éléments d'une interface utilisateur fournissent des informations sur leur usage ou sur leur signification grâce à des indications visuelles. Par exemple, une application de prise de note utilise un élément contenant l'image d'un signe "plus"  pour indiquer que l'utilisateur peut ajouter une nouvelle note. Une zone de saisie peut disposer d'une indication sous forme d'image qui permet de préciser l'information que l'utilisateur doit renseigner. Un utilisateur aveugle ne peut pas percevoir ces indications visuelles. Il est donc nécessaire d'employer un moyen pour ajouter une alternative à ces indications. Pour cela il faut utiliser les attributs `accessibilityLabel` et `accessibilityHint` dans la définition d'un élément d'interface. Les logiciels d'assistance se basent sur ces attributs pour communiquer aux utilisateurs les informations nécessaires à la bonne compréhension des éléments d'interface. Lorsque la langue des descriptions change, il est nécessaire d'utiliser l'attribut `accessibilityLanguage` en conséquence pour que le texte soit correctement prononcé par VoiceOver.

Exemple&nbsp;:
```
  [e1 setAccessibilityLabel:@"Ouvrir"];
  [e2 setAccessibilityLabel:@"Open"];
  [e2 setAccessibilityLanguage:@”en_US”];
```

NB&nbsp;: les noms propres, les mots d'origine étrangère présent dans le dictionnaire et les mots d'origine étrangère d'usage courant ("play", "OK", etc.) ne doivent pas faire l'objet d'un changement de langue.


## Identifier les caractéristiques des éléments d'interface

Les caractéristiques permettent de fournir des informations précises aux outils d'assistance sur le comportement et le rôle des éléments d'interface.

Les éléments de base d'UIKit sont déjà paramétrés avec les caractéristiques appropriées. Si un élément est personnalisé ou lors de la création d'une vue personnalisée, il est nécessaire de déclarer ses caractéristiques. Un élément peut disposer d'une ou de plusieurs caractéristiques qui seront combinées avec l'opérateur `OR`.

L'API d'accessibilité propose deux catégories de caractéristiques&nbsp;:

* Celles qui décrivent le type d'un élément (bouton, image, etc.)
* Celles qui définissent le comportement d'un élément (champ de recherche, lancer la lecture d'un son, etc.)

Les caractéristiques fournies par l'API sont notamment les suivantes&nbsp;:

* `UIAccessibilityTraitNone`&nbsp;: l'élément n'a pas de comportement défini. Cette caractéristique permet de supprimer toutes les autres (elle ne peut être utilisée en conjonction avec d'autres).
* `UIAccessibilityTraitButton`&nbsp;: l'élément doit être considéré comme un bouton. Le mot "bouton" sera prononcé par VoiceOver après le "label" de l'élément.
* `UIAccessibilityTraitLink`&nbsp,: l'élément doit être considéré comme un lien. Le mot "lien" sera prononcé par VoiceOver après le "label" de l'élément. Cette caractéristique doit être utilisée en lieu et place de `UIAccessibilityTraitButton` pour les actions qui ouvrent une page dans un navigateur.
* `UIAccessibilityTraitSearchField`&nbsp,;: l'élément doit être considéré comme un champ de saisie permettant d'effectuer une recherche. Cette caractéristique s'applique aux recherches restreintes au contexte de l'application. Par exemple, le champ de saisie d'un moteur de recherche sur le Web ne devra pas posséder cette caractéristique.
* `UIAccessibilityTraitImage`&nbsp,;: l'élément doit être considéré comme une image. Le mot "image" sera prononcé par VoiceOver après le "label" de l'élément. Cette caractérisitque peut être combinée avec d'autres (`UIAccessibilityTraitButton` ou `UIAccessibilityTraitLink` par exemple). `UIAccessibilityTraitImage` doit être utilisée lorsque l'apparence de l'image véhicule une information essentielle à la compréhension&nbsp;: elle ne doit pas être utilisée pour les images de décoration. Pour les boutons intégrés sous forme d'une image dont l'apparence n'apporte pas d'information essentielle à la compréhension de l'action, seule `UIAccessibilityTraitButton` devra être utilisée.
* `UIAccessibilityTraitSelected`&nbsp;: l'élément doit être considéré comme sélectionné. Le mot "sélectionné" sera prononcé par VoiceOver après le "label" de l'élément. Cette caractéristique pourra par exemple être utilisée pour indiquer l'état d'un panneau dans un système d'onglets (<span lang="en">tabpanel</span>).
* `UIAccessibilityTraitPlaysSound`&nbsp;: l'élément permet de lancer la lecture d'un son.
* `UIAccessibilityTraitKeyboardKey`&nbsp;: l'élément se comporte comme une touche de clavier.
* `UIAccessibilityTraitStaticText`&nbsp;: l'élément doit être considéré comme du texte statique, qui ne peut pas être modifié.
* `UIAccessibilityTraitSummaryElement`&nbsp;: représente un résumé de l'état courant d'un élément restitué lorsque l'application démarre. (NB&nbsp;: le focus n'est pas placé sur cet élément&nbsp;; le résumé n'est restitué qu'une fois à l'ouverture de l'application, mais pas à chaque nouvel affichage de la page concernée). Cette caractéristique peut ainsi être utilisée pour attirer l'attention de l'utilisateur de VoiceOver sur un élément important de la page d'accueil qui n'apparaît pas immédiatement dans l'ordre des éléments focusables.
* `UIAccessibilityTraitNotEnabled`&nbsp;: indique qu'un élément est désactivé ou n'est pas destiné à répondre aux interactions. Cette caractéristique peut par exemple être utilisée dans le cas où un bouton appraît dans l'interface mais dont le fonctionnement est temporairement désactivé. `UIAccessibilityTraitNotEnabled` peut être combinée avec d'autres caractéristiques (comme `UIAccessibilityTraitButton`).
* `UIAccessibilityTraitUpdatesFrequently`&nbsp;: indique que le contenu de l'élément est mis à jour trop fréquemment pour qu'il soit pertinent d'envoyer des notifications aux logiciels d'assistance à chaque changement. Cela permet d'éviter de "saturer" le contenu restitué par VoiceOver lorsqu'un élément change très rapidement (le contenu affiché par un chronomètre par exemple).
* `UIAccessibilityTraitStartsMediaSession`&nbsp;: indique que l'élément permet de lancer la lecture d'un contenu multimédia et ainsi d'indiquer à VoiceOver qu'il ne doit pas utiliser la synthèse vocale (la lecture du média pourra ainsi être lancée sans être perturbée par un texte énoncé par VoiceOver).
* `UIAccessibilityTraitAdjustable`&nbsp;: indique que la valeur d'un élément (comme un <span lang="en">slider</span>) peut être ajustée. Le mot "ajustable" sera prononcé par VoiceOver après le label, ainsi que le message "glissez vers le haut ou vers le bas pour modifier la valeur". L'utilisation de cette caractéristique provoque une modification du comportement par défaut de VoiceOver, qui redéfinit les gestes de "glissement" vers le haut et vers le bas pour ajuster la valeur de l'élément&nbsp;: l'action par défaut de ces gestes (par exemple la navigation de titre en titre) n'est donc plus disponible dans ce contexte..
* `UIAccessibilityTraitAllowsDirectInteraction`&nbsp;: représente un élément avec lequel l'utilisateur peut interagir directement via l'écran tactile. Lorsque cette caractéristique est activée, VoiceOVer n'interprète plus les gestes de l'utilisateur et laisse cette interprétation à la charge de l'application.
* `UIAccessibilityTraitCausesPageTurn`&nbsp;: représente un élément qui provoque automatiquement le passage à la page suivante lorsque les outils d'assistance arrivent à la fin du contenu. L'utilisation de cette caractéritique est par exemple pertinente dans une applicaiton de lecture de livre numérique pour automatiser le passage d'une page à la suivante lors de la restitution du texte par VoiceOver, sans que l'utilisateur n'ait à intervenir manuellement.
* `UIAccessibilityTraitHeader`&nbsp;: indique que l'élément possédant cette caractéristique permet de diviser du contenu (titre, barre de navigation, etc.). Son utilisation permet à VoiceOver de faciliter la navigation à l'intérieur de l'application.

Les caractéristiques peuvent être définies lors de l'initialisation d'une vue&nbsp;:
```
   - (id)init
   {
     _view = [[VuePers alloc] initWithFrame:CGRectZero];
     ...
     [_view setAccessibilityTraits:UIAccessibilityTraitButton | UIAccessibilityTraitPlaysSound];
     ...
   }
```

Les caractéristiques peuvent aussi être initialisées en définissant la méthode `accessibilityTraits`&nbsp;:
```
   - (UIAccessibilityTraits)accessibilityTraits
   {
      return UIAccessibilityTraitButton | UIAccessibilityTraitPlaysSound;
   }
```

## Définir la zone de restitution d'un élément

Il est possible de délimiter la zone de restitution d'un composant, c'est-à-dire la zone à l'intérieur de laquelle une interaction tactile déclenchera la restitution du contenu de ce composant par VoiceOver. Ceci peut être utile pour limiter le risque de confondre l'élément avec des éléments voisins proches lors de l'exploration tactile de l'écran. Pour cela, il faut utiliser la fonction `setAccessibilityFrame`.

Exemple&nbsp;:
```
  [_c setAccessibilityFrame:CGRectMake(10,10,130,130)];
```

## Mettre à jour les descriptions et caractéristiques

Lorsque les éléments d'interface d'une application changent, il est nécessaire de mettre à jour leurs descriptions et caractéristiques. Les méthodes permettant d'indiquer les attributs utiles pour l'accessibilité (`accessibilityLabel`, `accessibilityTraits`, etc.) doivent donc retourner des valeurs en fonction du contexte précis de l'élément et non "initialisées une fois pour toutes".

## Obtenir des informations sur l'état des options d'accessibilité

iOS met à disposition des méthodes permettant de récupérer des informations concernant l'état de certaines options d'accessibilité afin d'ajuster le comportement du programme en conséquence.

Par exemple&nbsp;:

* `UIAccessibilityIsVoiceOverRunning`&nbsp;: indique si VoiceOver est actif.
* `UIAccessibilityIsMonoAudioEnabled`&nbsp;: indique si le système audio est en mode mono.
* `UIAccessibilityIsClosedCaptioningEnabled`&nbsp;: indique si le sous-titrage est activé.



## Envoyer des notifications aux outils d'assistance

Lorsqu'un élément d'interface change, il est nécessaire d'envoyer une notification afin que les outils d'assistance soient informés de la modification. Des notifications doivent être envoyées lorsqu'un changement "significatif" se produit&nbsp;: il faut éviter de surcharger les outils d'assistance d'informations qu'ils ne seraient pas en mesure de restituer en temps réel.

L'envoi de notifications destinées aux outils d'assistance est réalisé avec la méthode `UIAccessibilityPostNotification`.

Par exemple, pour notifier le changement d'un élément d'interface&nbsp;:
```
  UIAccessibilityPostNotification(UIAccessibilityLayoutChangedNotification, nil);
```

`UIAccessibility` fournit des notifications qui peuvent être envoyées par les applications, parmi lesquelles&nbsp;:

* `UIAccessibilityLayoutChangedNotification`&nbsp;: permet d'envoyer une "annonce" destinée à être restituée par les outils d'assistance (un message peut être passé via une chaîne de caractère en paramètre). Cette notification doit être utilisée pour avertir les utilisateurs de VoiceOver du changement d'un élément de l'interface.
* `UIAccessibilityAnnouncementNotification`&nbsp;: permet d'envoyer une "annonce" destinée à être restituée par les outils d'assistance (un message peut être passé via une chaîne de caractère en paramètre). Cette notification doit être utilisée pour avertir les utilisateurs d'un événement qui n'a pas modifié d'élément d'interface ou qui en a modifié pour une durée brève.
* `UIAccessibilityPageScrolledNotification`&nbsp;: permet de fournir de l'information sur le contenu de l'écran après qu'un utilisateur ait "scrollé" avec VoiceOver.
* `UIAccessibilityPauseAssistiveTechnologyNotification` et `UIAccessibilityResumeAssistiveTechnologyNotification`&nbsp;: permet de stopper temporairement et de recommencer les actions réalisées par les outils d'assistance spécifiés en paramètre.
* `UIAccessibilityScreenChangedNotification`&nbsp;: permet de notifier les outils d'assistance lors de l'apparition d'une nouvelle vue qui prend une part importante de l'écran. Une chaîne qui sera restituée par les outils d'assistance ou bien un élément d'interface sur lequel sera positionné le focus de l'outil d'assistance utilisé peuvent être passés en paramètre.

## Capturer et traiter les notifications envoyées par UIKit

UIKit envoie des notifications relatives à l'accessibilité qu'il peut être utile de traiter pour améliorer le comportement de l'application. Capturer ces notifications permet de personnaliser le comportement de l'application pour les utilisateurs d'outils d'assistance.

Ces notifications peuvent être capturée via le centre de notifications. Par exemple, pour capturer la notification `UIAccessibilityVoiceOverStatusChanged`&nbsp;:
```
[[NSNotificationCenter defaultCenter]
    addObserver:self
    selector:@selector(updateUserInterfaceForVoiceOver:)
    name:UIAccessibilityVoiceOverStatusChanged
    object:nil];
```

Les notifications relatives à l'accessibilité fournies par l'API sont notamment les suivantes&nbsp;:

* `UIAccessibilityVoiceOverStatusChanged`&nbsp;: envoyée par UIKit lorsque VoiceOver démarre ou est arrêté.
* `UIAccessibilityAnnouncementDidFinishNotification`&nbsp;: envoyée par UIKit lorsque la restitution d'une annonce par les outils d'assistance est terminée. Cette notification renvoie un "dictionnaire" contenant deux clés, `UIAccessibilityAnnouncementKeyStringValue` et `UIAccessibilityAnnouncementKeyWasSuccessful`, qui permet de connaître le message qui a été restitué et de savoir si la restitution s'est terminée avec succès ou non.

## Quelques cas particuliers

### Les modales

Les vues d'UIKit qui permettent de mettre en oeuvre des modales (`UIAlertView` par exemple) sont accessibles&nbsp;: elles permettent à l'utilisateur d'outils d'assistance de ne pas interagir avec le contenu situé en dehors de la vue. Pour les vues personnalisées, il faut utiliser l'attribut `accessibilityViewIsModal`&nbsp;: lorsque cet attribut vaut "YES" les éléments situés à l'intérieur des vues voisines seront ignorées par les outils d'assistance.

### Les vues tableau (TableView)

Les cellules d'une vue en tableau sont considérées automatiquement comme des containers&nbsp;: il n'est donc pas nécessaire d'implémenter le protocole `UIAccessibilityContainer` pour déclarer la liste des éléments contenus.
Pour faciliter la restitution par les outils d'assistance, il peut être utile de grouper les informations contenues dans plusieurs cellules en une seule description afin de minimiser le nombre d'opérations que devra effectuer l'utilisateur pour y accéder. Par exemple, il est pertinent de regrouper l'intitulé d'un produit, son poids et son prix en un texte qui sera restitué à l'utilisateur en une seule fois, sans qu'il n'ait à réaliser des gestes pour accéder à chacun des éléments séparément.

La méthode `accessibilityLabel` pourra alors être définie de la sorte&nbsp;:
```
  - (NSString *)accessibilityLabel
  {
    NSString *prodNomText = [self.prodNom accessibilityLabel];
    NSString *prodPoidsText = [self.prodPoids accessibilityLabel];
    NSString *prodPrixText = [self.prodPrix accessibilityLabel];

    return [NSString stringWithFormat:@"%@, %@, %@", prodNomText, prodPoidsText, prodPrixText];
}
```


## Guides connexes

Les guides suivants peuvent être consultés en complément&nbsp;:

* [Guide d'audit d'applications mobiles](https://github.com/DISIC/guide-mobile_app_audit)
* [Guide de conception d'applications mobiles accessibles](https://github.com/DISIC/guide-mobile_app_conception)
* [Guide de développement d'applications mobiles accessibles avec Ionic et OnsenUI](https://github.com/DISIC/guide-mobile_app_dev_hybride)

## Ressources externes et références

Sources&nbsp;:

* <a href="https://developer.apple.com/library/ios/documentation/UIKit/Reference/UIKit_Framework/" lang="en">UIKit Framework Reference</a>
* <a href="https://developer.apple.com/accessibility/ios/" lang="en">Accessibility for iOS Developers</a>
* <a href="http://www.bbc.co.uk/guidelines/futuremedia/accessibility/mobile" lang="en">BBC Mobile Accessibility Prototype</a>

## Licence
Ce document est la propriété du Secrétariat général à la modernisation de l'action publique français (SGMAP). Il est placé sous la [licence ouverte 1.0 ou ultérieure](http://wiki.data.gouv.fr/wiki/Licence_Ouverte_/_Open_Licence), équivalente à une licence <i lang="en">Creative Commons BY</i>. Pour indiquer la paternité, ajouter un lien vers la version originale du document disponible sur le [compte <span lang="en">Github</span> de la DInSIC](https://github.com/DISIC).


