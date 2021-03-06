---

copyright:
  years: 2016

---

#Configuration d'une authentification personnalisée pour les applications Web {{site.data.keyword.amashort}}
{: #custom-web}

Dernière mise à jour : 21 juillet 2016
{: .last-updated}

Ajoutez une authentification personnalisée et une fonctionnalité de sécurité {{site.data.keyword.amashort}} à votre application Web.

## Avant de commencer
{: #before-you-begin}
Avant de commencer, vous devez disposer des éléments suivants :

*	Une application Web. 
*	Une instance d'une application {{site.data.keyword.Bluemix_notm}} qui est protégée par le service {{site.data.keyword.amashort}}. Pour plus d'informations sur la création d'une application back end {{site.data.keyword.Bluemix_notm}}, voir [Initiation à {{site.data.keyword.amashort}}](https://console.{DomainName}/docs/services/mobileaccess/getting-started.html).
*	L'URI de redirection finale (à l'issue du processus d'autorisation).


Pour plus d'informations, voir :
 * [Initiation à {{site.data.keyword.amashort}}](https://console.{DomainName}/docs/services/mobileaccess/getting-started.html)
 * [Utilisation d'un fournisseur d'identité personnalisé](https://console.{DomainName}/docs/services/mobileaccess/custom-auth.html)
 * [Création d'un fournisseur d'identité personnalisé](https://console.{DomainName}/docs/services/mobileaccess/custom-auth-identity-provider.html)
 * [Configuration de {{site.data.keyword.amashort}} pour l'authentification personnalisée](https://console.{DomainName}/docs/services/mobileaccess/custom-auth-config-mca.html)


##Configuration d'un fournisseur d'identité personnalisé 

Lors de la création d'un fournisseur d'identité personnalisé, vous devez définir une méthode POST avec une route sous la structure suivante : 

`/apps/:tenantID/<nom_de_votre_domaine>/handleChallengeAnswer`

`tenantID` est un paramètre d'URL et `<nom_de_votre_domaine>` le nom de domaine de votre choix. 

Le corps de la demande contiendra un objet `challengeAnswer` hébergeant les éléments `username`
et `password`.
Après la validation de l'utilisateur, cette route doit renvoyer un objet JSON avec la structure suivante. 


```json
{ 
            status: "success", 
            userIdentity: { 
                userName: <nom_utilisateur>,
                displayName: <nom_d'affichage>
                attributes: <autres_attributs_json>
            } 
        } 

 ```

**Remarque :** La zone `attributes` est facultative. 

Le code suivant illustre une demande Post de ce type.

```Java
var app = express();
var bodyParser = require('body-parser');
app.use(bodyParser.json());

var users = {
    "John": {
      password: "123",
      displayName: "John Doe"
    }
};

app.post('/apps/:tenantID/customAuthRealm_1/handleChallengeAnswer',
         function(req, res) {
         console.log ("tenantID " + req.params.tenantID);
         
         var challengeAnswer = req.body.challengeAnswer;
         console.log ("challengeAnswer " + JSON.stringify(challengeAnswer));

         if (challengeAnswer && users[challengeAnswer.username] && challengeAnswer.password === users[challengeAnswer.username].password) {
         res.json({
                  status: "success",
                  userIdentity: {
                  userName: challengeAnswer.username,
                  displayName: users[challengeAnswer.username].displayName
                  }
                  });
         } else {
         res.json({
                  status: "failure"
                  });
         }
         
         });

```


##Configuration de {{site.data.keyword.amashort}} pour l'authentification personnalisée 

Une fois que vous avez configuré votre fournisseur d'identité personnalisé, vous pouvez activer l'authentification personnalisée dans le tableau de bord
{{site.data.keyword.amashort}} . 

1. Ouvrez le tableau de bord {{site.data.keyword.Bluemix_notm}}. 
2. Cliquez sur la vignette de l'application {{site.data.keyword.amashort}} pertinente. Le tableau de bord de l'application est chargé. 
3. Cliquez sur le bouton **Configurer** sur la vignette Personnalisé. 
4. Dans la zone de texte **Nom de domaine**, entrez le nom de domaine configuré dans votre noeud final de gestionnaire de fournisseur
d'identité personnalisé. t
5. Entrez l'URL du fournisseur d'identité personnalisé. 
6. Entrez l'URI de redirection d'application Web à utiliser par le tableau de bord {{site.data.keyword.amashort}} après l'aboutissement de
l'authentification. 
7. Enregistrez. 


##Implémentation du flux d'autorisation {{site.data.keyword.amashort}} à l'aide d'un fournisseur d'identité personnalisé 

La variable d'environnement `VCAP_SERVICES` est créée automatiquement pour chaque instance de service {{site.data.keyword.amashort}} et contient les propriétés requises pour le processus d'autorisation. Elle est constituée d'un objet JSON et peut être affichée en cliquant sur **Variables d'environnement** dans la barre de navigation sur le côté gauche de votre application.

Pour demander l'autorisation de l'utilisateur, redirigez le navigateur vers le noeud final du serveur d'autorisation. Pour ce faire, procédez comme suit : 

1. Identifiez le noeud final d'autorisation (`authorizationEndpoint`) et l'ID client
(`clientId`) depuis les données d'identification du service stockées dans la variable d'environnement
`VCAP_SERVICES`. 

  **Remarque :** si vous avez ajouté le service {{site.data.keyword.amashort}} dans votre application avant l'ajout de la prise en charge Web, il se peut que vous n'ayez pas de noeud final de jeton dans les données d'identification pour le service. A la place, utilisez les URL suivantes, selon votre région {{site.data.keyword.Bluemix_notm}} : 
 
  Sud des Etats-Unis : 
  ```
  https://mobileclientaccess.ng.bluemix.net/oauth/v2/authorization 
  ```
  Londres : 
  ```
  https://mobileclientaccess.eu-gb.bluemix.net/oauth/v2/authorization 
  ```
  Sydney : 
  ```
  https://mobileclientaccess.au-syd.bluemix.net/oauth/v2/authorization 
  ```
2. Construisez l'URI du serveur d'autorisation en utilisant `response_type("code")`,
`client_id` et `redirect_uri` en tant que paramètres de requête.  
1. Redirigez l'utilisateur depuis votre application Web vers l'URI généré. 

L'exemple suivant extrait les paramètres depuis la variable `VCAP_SERVICES`,
construit l'URL et envoie la demande de redirection.

 ```Java
var cfEnv = require("cfenv"); 
app.get("/protected", checkAuthentication, function(req, res, next){ 
  res.send("Bonjour, ceci est un noeud final protégé");
  }
); 


function checkAuthentication(req, res, next){ 
  // Vérifie si l'utilisateur est authentifié
  if (req.session.userIdentity){ 
    next() 
  } else { 
    // If not - redirect to authorization server 
    var mcaCredentials = cfEnv.getAppEnv().services.AdvancedMobileAccess[0].credentials; 
    var authorizationEndpoint = mcaCredentials.authorizationEndpoint; 
    var clientId = mcaCredentials.clientId; 
    var redirectUri = "http://some-server/oauth/callback"; // Your web application redirect uri 
    var redirectUrl = authorizationEndpoint + "?response_type=code";
    redirectUrl += "&client_id=" + clientId; 
    redirectUrl += "&redirect_uri=" + redirectUri; 
    res.redirect(redirectUrl); 
  } 

} 

 ```
 
Notez que le paramètre `redirect_uri` représente l'URI de redirection de votre application Web
et qu'il doit être égal à celui défini dans le tableau de bord {{site.data.keyword.amashort}}.  

Un paramètre `state` peut être transmis de pair avec la requête. Ce paramètre sera propagé à la méthode POST
du fournisseur d'identité personnalisé et est accessible depuis le corps de la requête (`req.body.stateId`).  

Après redirection au noeud final d'autorisation, l'utilisateur accède à un formulaire de connexion. Une fois que les données d'identification de
l'utilisateur ont été authentifiées par le fournisseur d'identité personnalisé, le service {{site.data.keyword.amashort}} appelle l'URI de redirection
de votre service Web en soumettant le code d'accord en tant que paramètre de requête.  

Après la redirection, l'utilisateur reçoit un formulaire de connexion. Une fois les données d'identification de l'utilisateur authentifiées par le
fournisseur d'identité personnalisé, le service {{site.data.keyword.amashort}} appelle l'URI de redirection de l'application Web en soumettant le
code d'accord en tant que paramètre  de la requête. 

##Obtention des jetons

L'étape suivante consiste à obtenir le jeton d'accès et le jeton d'identité à l'aide du code d'accord reçu auparavant. Pour ce faire, procédez comme suit : 

1. Extrayez les éléments `authorizationEndpoint`, `clientId` et `secret` depuis les données
d'identification du service stockées dans la variable d'environnement `VCAP_SERVICES`. 

   **Remarque :** si vous avez ajouté le service {{site.data.keyword.amashort}} dans votre application avant l'ajout de la prise en charge Web, il se peut que vous n'ayez pas de noeud final de jeton dans les données d'identification pour le service. A la place, utilisez les URL suivantes, selon votre région {{site.data.keyword.Bluemix_notm}} : 

 Sud des Etats-Unis : 
 ```
     https://mobileclientaccess.ng.bluemix.net/oauth/v2/token   
 ```
 Londres : 
 ```
     https://mobileclientaccess.eu-gb.bluemix.net/oauth/v2/token
 ``` 
 Sydney : 
 ``` 
     https://mobileclientaccess.au-syd.bluemix.net/oauth/v2/token 
 ```
1. Envoyez une requête POST à l'URI du serveur de jeton en spécifiant les éléments
`grant_type`, `client_id`, `redirect_uri` et
`code` sous forme de paramètres de masque et les éléments `clientId` et `secret` en tant que données
d'authentification HTTP de base.


Dans l'exemple suivant, le code extrait les valeurs requises et les envoie dans une requête POST.


 ```Java var cfEnv = require("cfenv"); var base64url = require("base64url "); app.get("/oauth/callback", function(req, res, next){ 
    var mcaCredentials = cfEnv.getAppEnv().services.AdvancedMobileAccess[0].credentials;
    var tokenEndpoint = mcaCredentials.tokenEndpoint; 

    var formData = { 
      grant_type: "authorization_code",
      client_id: mcaCredentials.clientId,
      redirect_uri: "http://some-server/oauth/callback",   // URI de redirection d'application Web
      code: req.query.code
    }

  request.post({ 
    url: tokenEndpoint,
    formData: formData
    }, function (err, response, body){ 
      var parsedBody = JSON.parse(body); 

      req.session.accessToken = parsedBody.access_token;
      req.session.idToken = parsedBody.id_token;
      var idTokenComponents = parsedBody.id_token.split("."); // [en-tête, contenu, signature]
      var decodedIdentity= base64url(idTokenComponents[1]);
      req.session.userIdentity = JSON.parse(decodedIdentity)["imf.user"];
      res.redirect("/");
    }
    ).auth(mcaCredentials.clientId, mcaCredentials.secret);
  }
); 

 ```
Notez que le paramètre `redirect_uri` doit correspondre au paramètre `redirect_uri` utilisé auparavant dans la demande
d'autorisation. La valeur du paramètre code doit être le code d'accord reçu dans la réponse à l'issue de la demande d'autorisation. Ce code n'est valide que
pendant 10 minutes, après quoi vous devrez en obtenir un nouveau.

Le corps de la réponse contiendra les éléments `access_token` et `id_token` au format JWT (https://jwt.io/).

Une fois que vous avez reçu les jetons d'accès et d'identité, vous pouvez marquer la session Web comme authentifiée
et, si vous le désirez, rendre persistants ces jetons.

##Utilisation du jeton d'accès et du jeton d'identité obtenus 

Le jeton d'identité contient des informations sur l'identité de l'utilisateur. Dans le cas d'une authentification personnalisée, le jeton contiendra toutes les informations
renvoyées par le fournisseur d'identité personnalisé lors de l'authentification. Sous la zone `imf.user`, la zone
`displayName` contiendra le nom d'affichage (`displayName`) renvoyé par le fournisseur d'identité personnalisé
et la zone `id` contiendra le nom d'utilisateur (`userName`).  Toutes les autres valeurs renvoyées par le fournisseur d'identité
personnalisé sont renvoyées dans la zone `attributes` sous `imf.user`.  

Le jeton d'accès permet une communication avec les ressources protégées par les filtres d'autorisation de {{site.data.keyword.amashort}} (voir [Protection des ressources](protecting-resources.html)). Pour soumettre des demandes à des ressources protégées, ajoutez aux demandes un en-tête Authorization doté de la structure suivante : 

`Authorization=Bearer <jeton_accès> <jeton_ID>` 

####Conseils : 
{: #tips_token}

* Les éléments `<accessToken>` et `<idToken>` doivent être séparés par un espace.

* Le jeton d'identité est facultatif. Si vous l'omettez, il est possible d'accéder à la ressource protégée, mais sans recevoir
d'informations sur l'utilisateur autorisé. 



