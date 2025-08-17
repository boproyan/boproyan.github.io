+++
title = 'Créer une API RESTful sécurisée avec CodeIgniter et des JWT'
date = 2021-12-30 00:00:00 +0100
categories = ['php', 'authentification']
+++
  - [API RESTful avec CodeIgniter et JWT](#api-restful-avec-codeigniter-et-jwt)
    - [Prérequis](#prérequis)
    - [Mise en route](#mise-en-route)
    - [Variables environnement](#variables-environnement)
    - [Migrations et seeders](#migrations-et-seeders)
    - [Modèles d'entité](#modèles-dentité)
    - [Implémentation de JWT](#implémentation-de-jwt)
    - [Création d'un JWT Helper](#création-dun-jwt-helper)
    - [Création filtre d'authentification](#création-filtre-dauthentification)
    - [Enregistrement](#enregistrement)
    - [Authentification](#authentification)
    - [Validation de l'utilisateur](#validation-de-lutilisateur)
    - [Création d'un contrôleur client](#création-dun-contrôleur-client)
  - [curl](#curl)
  - [API ready](#api-ready)
    - [Lancer le serveur](#lancer-le-serveur)
    - [Ajout d'un token d'accès](#ajout-dun-token-daccès)
    - [Création d'un nouveau client](#création-dun-nouveau-client)
    - [Récupération de la liste de tous les clients](#récupération-de-la-liste-de-tous-les-clients)
    - [Client par id](#client-par-id)
    - [Mise à jour d'un client existant](#mise-à-jour-dun-client-existant)
    - [Suppression d'un client existant](#suppression-dun-client-existant)
    - [Conclusion](#conclusion)
  - [Codeigniter Nginx server configuration](#codeigniter-nginx-server-configuration)
    - [Web Server Site Configuration](#web-server-site-configuration)
      - [Recommended Nginx Configuration](#recommended-nginx-configuration)
    - [Sub Directory Site Application](#sub-directory-site-application)
      - [Application BaseUrl](#application-baseurl)
      - [Nginx Configuration](#nginx-configuration)

---

## API RESTful avec CodeIgniter et JWT

* Article original : [Create a Secured RESTful API with CodeIgniter and JSON Web Tokens](https://www.twilio.com/blog/create-secured-restful-api-codeigniter-php)  
* [CodeIgniter 4 Login and Register with JWT (JSON Web Token)](https://mfikri.com/en/blog/codeigniter-login-jwt)
* [Nginx - Codeigniter](https://www.nginx.com/resources/wiki/start/topics/recipes/codeigniter/)
* [Codeigniter 3 server configuration for Nginx & Apache](https://gist.github.com/yidas/30a611449992b0fac173267951e5f17f)
* [PHP Authorization with JWT (JSON Web Tokens)](https://www.sitepoint.com/php-authorization-jwt-json-web-tokens/)
    * <https://github.com/sitepoint-editors/basic-php-jwt-auth-example>


*L'utilisation et les applications croissantes des services cloud nécessitent un style architectural plus efficace que le protocole SOAP (Simple Object Access Protocol). REST (REpresentational State Transfer) permet une communication légère et sans état entre les clients et l'interface de programmation d'applications (API). La communication étant sans état, le contrôle d'accès des API RESTful est basé sur des tokens qui transportent suffisamment d'informations pour déterminer si le client est autorisé à effectuer l'action requêtée sur la ressource.*

*CodeIgniter est un puissant framework PHP avec un encombrement très faible qui permet aux développeurs de construire des applications Web complètes.*

### Prérequis

* PHP8.0
* [Composer](https://www.hostinger.fr/tutoriels/comment-installer-et-utiliser-composer/) utilisé pour la gestion des dépendances dans votre projet CodeIgniter
* Postman est un outil permettant de manipuler une API depuis une interface graphique
télécharger la version native de PostMan correspondant à sa plateforme, à savoir x86 ou x64, depuis l’adresse https://www.getpostman.com/apps
décompressé dans le répertoire /opt : `sudo tar -xzf Postman-linux-x86_64-9.6.2.tar.gz -C /opt`
mettre l’application PostMan dans le chemin des exécutables : `sudo ln -s /opt/Postman/Postman /usr/local/bin/postman`  
* Une instance de base de données locale MySQL 

Pour démontrer comment créer une API CodeIgniter sécurisée, nous allons créer une API qui sera utilisée pour gérer la base de données client d'une entreprise. Cette base de données contient les données suivantes sur chaque client :

    Nom
    Adresse e-mail
    Applications ou commentaire

L'API obtenue à la fin de ce tutoriel présentera les fonctionnalités suivantes :

1.    Enregistrer un nouvel utilisateur
2.    Authentifier un utilisateur existant
3.    Ajouter un nouveau client
4.    Modifier les détails d'un client existant
5.    Afficher tous les clients
6.    Afficher un seul client par ID
7.    Supprimer un seul client par ID

Les fonctions 3 à 7 seront limitées aux utilisateurs authentifiés.

### Mise en route

Créez un nouveau projet CodeIgniter à l'aide de Composer.

    composer create-project codeigniter4/appstarter ci-secure-api

Ceci créera un nouveau projet CodeIgniter dans un dossier nommé `ci-secure-api`. Une fois l'installation terminée, accédez au dossier de projet nouvellement créé à partir du terminal et exécutez l'application sur le serveur de développement local fourni avec CodeIgniter. Pour ce faire, utilisez la commande suivante :

    cd ci-secure-api && php spark serve

```
CodeIgniter v4.1.5 Command Line Tool - Server Time: 2021-12-27 07:35:48 UTC-06:00

CodeIgniter development server started on http://localhost:8080
Press Control-C to stop.
[Mon Dec 27 13:35:48 2021] PHP 8.0.14 Development Server (http://localhost:8080) started
```

L'appel http se fait depuis le poste archlinux qui héberge le debian 11 virtuel (bullseyes)  
Ajouter `192.168.0.130 ouestyan` au fichier `/etc/hosts` du poste archlinux  

Accéder http://localhost:8080 via le navigateur du poste archlinux  
On utilise la redirection port SSH

Vérification,ouvrir un terminal sur le client linux qui dispose des clés ssh et lancer la commande

    ssh -L 9000:localhost:8080 bullsadmin@192.168.0.130 -p 55130 -i /home/yann/.ssh/vm-bullseyes

http://localhost:9000  
![](api_restfull_001.png){:width=600}

### Variables environnement

Maintenant que CodeIgniter est installé et en cours d'exécution, l'étape suivante consiste à fournir des variables d'environnement qui seront utilisées par notre application.  
Arrêtez l'exécution de l'application en appuyant sur les touches CTRL + C du clavier et effectuez une copie du fichier .env nommé .env à l'aide de la commande ci-dessous :

    cp env .env
    nano .env

CodeIgniter démarre en mode production par défaut. Dans le cadre de ce tutoriel, nous allons le passer en mode développement. Pour ce faire, annuler le commentaire de la ligne ci-dessous et définissez-la sur development :

    CI_ENVIRONMENT = development

Ensuite, créez une base de données dans votre environnement local  

    mysql -uroot -pRenvoieFavoriFoulonIngambeParmi -e "CREATE DATABASE dbapi; GRANT ALL ON dbapi.* TO 'dbapi'@'localhost' IDENTIFIED BY 'GliomeResteCadranMarmite'; FLUSH PRIVILEGES;"


et supprimez le commentaire des variables suivantes pour mettre à jour chaque valeur et établir une connexion réussie à la base de données :

```
database.default.hostname = localhost
database.default.database = dbapi
database.default.username = dbapi
database.default.password = GliomeResteCadranMarmite
database.default.DBDriver = MySQLi # this is the driver for a MySQL connection. There are also drivers available for postgres & SQLite3.
```

### Migrations et seeders

Maintenant que nous avons créé une base de données et configuré une connexion à celle-ci, nous allons créer des migrations pour les tables user et client. Les fichiers de migration sont généralement utiles pour créer une structure de base de données appropriée. Les migrations et les seeders seront créés à l'aide de l'outil [CLI CodeIgniter](https://codeigniter.com/user_guide/cli/index.html) avec la commande `php spark migrate:create`

L'interface de ligne de commande vous demandera de nommer le fichier de migration, après quoi le fichier de migration sera créé dans le répertoire `App/Database/Migrations`.  
Pour ce tutoriel, vous allez créer deux fichiers de migration nommés `add_client` et `add_user`


```
bullsadmin@bullseyes:~/ci-secure-api$ php spark migrate:create

CodeIgniter v4.1.5 Command Line Tool - Server Time: 2021-12-27 08:14:54 UTC-06:00

Migration class name : add_client

File created: APPPATH/Database/Migrations/2021-12-27-141637_AddClient.php

bullsadmin@bullseyes:~/ci-secure-api$ php spark migrate:create

CodeIgniter v4.1.5 Command Line Tool - Server Time: 2021-12-27 08:16:52 UTC-06:00

Migration class name : add_user

File created: APPPATH/Database/Migrations/2021-12-27-141704_AddUser.php
```

Le nom du fichier de migration sera précédé d'une séquence numérique au format de date **AAAA-MM-JJ-HHIISS**. Reportez-vous à la documentation CodeIgniter pour obtenir une explication plus détaillée.

Ensuite, mettez à jour le contenu du fichier de migration **add_client** (`app/Database/Migrations/2021-12-27-141637_AddClient.php`) comme suit

```php
<?php
use CodeIgniter\Database\Migration;

class AddClient extends Migration
{
    public function up()
    {
        $this->forge->addField([
            'id' => [
                'type' => 'INT',
                'constraint' => 5,
                'unsigned' => true,
                'auto_increment' => true,
            ],
            'name' => [
                'type' => 'VARCHAR',
                'constraint' => '100',
                'null' => false
            ],
            'email' => [
                'type' => 'VARCHAR',
                'constraint' => '100',
                'null' => false,
                'unique' => true
            ],
            'retainer_fee' => [
                'type' => 'INT',
                'constraint' => 100,
                'null' => false,
                'unique' => true
            ],
            'updated_at' => [
                'type' => 'datetime',
                'null' => true,
            ],
        'created_at datetime default current_timestamp',
        ]);
        $this->forge->addPrimaryKey('id');
        $this->forge->createTable('client');
    }

    public function down()
    {
        $this->forge->dropTable('client');
    }
}
```

Ici, nous avons spécifié les champs et les types de données correspondants pour la table Client.

Ouvrez ensuite le fichier de migration **add_user** (`app/Database/Migrations/2021-12-27-141704_AddUser.php`) et remplacez son contenu par ce qui suit :

```php
<?php

use CodeIgniter\Database\Migration;

class AddUser extends Migration
{
    public function up()
    {
        $this->forge->addField([
            'id' => [
                'type' => 'INT',
                'constraint' => 5,
                'unsigned' => true,
                'auto_increment' => true,
            ],
            'name' => [
                'type' => 'VARCHAR',
                'constraint' => '100',
                'null' => false
            ],
            'email' => [
                'type' => 'VARCHAR',
                'constraint' => '100',
                'null' => false,
                'unique' => true
            ],
            'password' => [
                'type' => 'VARCHAR',
                'constraint' => '255',
                'null' => false,
                'unique' => true
            ],
            'updated_at' => [
                'type' => 'datetime',
                'null' => true,
            ],
            'created_at datetime default current_timestamp',
        ]);
        $this->forge->addPrimaryKey('id');
        $this->forge->createTable('user');
    }

    public function down()
    {
        $this->forge->dropTable('user');
    }
}
```

Le contenu ci-dessus vous aidera à créer la table **user** et ses champs. Exécutez maintenant vos migrations à l'aide de la commande ci-dessous :

    php spark migrate

```
CodeIgniter v4.1.5 Command Line Tool - Server Time: 2021-12-27 08:43:03 UTC-06:00

Running all new migrations...
	Running: (App) 2021-12-27-141637_\AddClient
	Running: (App) 2021-12-27-141704_\AddUser
Done migrations.
```

Pour faciliter le développement, seedez des données client factices dans votre base de données. [L'ensemble factice fzaninotto](https://github.com/fzaninotto/Faker) est une dépendance par défaut dans le squelette CodeIgniter et peut être utilisé pour ajouter des clients aléatoires à la base de données. Tout comme pour la migration, l'outil CLI CodeIgniter sera utilisé pour créer un seeder pour les clients. Exécutez la commande suivante :

    php spark make:seeder

L'outil CLI requêtera le nom **ClientSeeder**. Un fichier *ClientSeeder.php* sera créé dans le répertoire App/Database/Seeds (`app/Database/Seeds/ClientSeeder.php`). Ouvrez le fichier et remplacez son contenu par ce qui suit :

```php
<?php

namespace App\Database\Seeds;

use CodeIgniter\Database\Seeder;
use Faker\Factory;

class ClientSeeder extends Seeder
{
    public function run()
    {
        for ($i = 0; $i < 10; $i++) { //to add 10 clients. Change limit as desired
            $this->db->table('client')->insert($this->generateClient());
        }
    }

    private function generateClient(): array
    {
        $faker = Factory::create();
        return [
            'name' => $faker->name(),
            'email' => $faker->email,
            'retainer_fee' => random_int(100000, 100000000)
        ];
    }
}
```

Peuplez la base de données avec des clients factices à l'aide de la commande suivante :

    php spark db:seed ClientSeeder

```
CodeIgniter v4.1.5 Command Line Tool - Server Time: 2021-12-27 08:44:06 UTC-06:00

Seeded: App\Database\Seeds\ClientSeeder
```

Accès contenu base

```
bullsadmin@bullseyes:~/ci-secure-api$ mysql -udbapi -pGliomeResteCadranMarmite dbapi -e "SELECT * FROM client"
+----+--------------------------+------------------------------+--------------+------------+---------------------+
| id | name                     | email                        | retainer_fee | updated_at | created_at          |
+----+--------------------------+------------------------------+--------------+------------+---------------------+
|  1 | Colt Hintz               | mackenzie72@hotmail.com      |     21526147 | NULL       | 2021-12-27 14:44:06 |
|  2 | Ms. Linda Romaguera DVM  | frederick.herzog@hotmail.com |     39778512 | NULL       | 2021-12-27 14:44:06 |
|  3 | Juston Effertz           | ansley07@sporer.biz          |     81860215 | NULL       | 2021-12-27 14:44:06 |
|  4 | Mrs. Melissa Auer        | maritza14@hotmail.com        |     94636348 | NULL       | 2021-12-27 14:44:06 |
|  5 | Giles Predovic           | mkunze@gleason.biz           |      1522348 | NULL       | 2021-12-27 14:44:06 |
|  6 | Henderson Keeling        | brannon23@sipes.org          |     15540299 | NULL       | 2021-12-27 14:44:06 |
|  7 | Walton Hettinger         | abarrows@kautzer.info        |     70656137 | NULL       | 2021-12-27 14:44:06 |
|  8 | Cydney Bernier IV        | elisa.dickens@torphy.com     |     53441851 | NULL       | 2021-12-27 14:44:06 |
|  9 | Dr. Kathleen Greenfelder | iwolf@boehm.com              |     11875923 | NULL       | 2021-12-27 14:44:06 |
| 10 | Fannie Turner            | ralph43@yahoo.com            |     60052006 | NULL       | 2021-12-27 14:44:06 |
+----+--------------------------+------------------------------+--------------+------------+---------------------+
bullsadmin@bullseyes:~/ci-secure-api$ mysql -udbapi -pGliomeResteCadranMarmite dbapi -e "SELECT * FROM user"
bullsadmin@bullseyes:~/ci-secure-api$ mysql -udbapi -pGliomeResteCadranMarmite dbapi -e "SELECT * FROM migrations"
+----+-------------------+------------+---------+-----------+------------+-------+
| id | version           | class      | group   | namespace | time       | batch |
+----+-------------------+------------+---------+-----------+------------+-------+
|  1 | 2021-12-27-141637 | \AddClient | default | App       | 1640616184 |     1 |
|  2 | 2021-12-27-141704 | \AddUser   | default | App       | 1640616184 |     1 |
+----+-------------------+------------+---------+-----------+------------+-------+
```

### Modèles d'entité

Pour l'interaction de l'API avec la base de données, le [modèle de CodeIgniter](https://codeigniter4.github.io/userguide/models/model.html#using-codeigniter-s-model) sera utilisé. Pour que cela fonctionne, deux modèles seront créés : un pour l'utilisateur et un autre pour le client.

Ouvrez le répertoire App/Models (`app/Models/`) et créez les fichiers suivants : 

Dans `app/Models/UserModel.php`, ajoutez ce qui suit :

```php
<?php

namespace App\Models;

use CodeIgniter\Model;
use Exception;

class UserModel extends Model
{
    protected $table = 'user';
    protected $allowedFields = [
        'name',
        'email',
        'password',
    ];
    protected $updatedField = 'updated_at';

    protected $beforeInsert = ['beforeInsert'];
    protected $beforeUpdate = ['beforeUpdate'];

    protected function beforeInsert(array $data): array
    {
        return $this->getUpdatedDataWithHashedPassword($data);
    }

    protected function beforeUpdate(array $data): array
    {
        return $this->getUpdatedDataWithHashedPassword($data);
    }

    private function getUpdatedDataWithHashedPassword(array $data): array
    {
        if (isset($data['data']['password'])) {
            $plaintextPassword = $data['data']['password'];
            $data['data']['password'] = $this->hashPassword($plaintextPassword);
        }
        return $data;
    }

    private function hashPassword(string $plaintextPassword): string
    {
        return password_hash($plaintextPassword, PASSWORD_BCRYPT);
    }
                                      
    public function findUserByEmailAddress(string $emailAddress)
    {
        $user = $this
            ->asArray()
            ->where(['email' => $emailAddress])
            ->first();

        if (!$user) 
            throw new Exception('User does not exist for specified email address');

        return $user;
    }
}
```

Les fonctions *beforeInsert* et *beforeUpdate* vous permettent d'effectuer une opération sur l'entité *User* avant de l'enregistrer dans la base de données.  
Dans ce cas, le mot de passe de l'utilisateur est hashé avant d'être enregistré dans la base de données.

Dans `app/Models/ClientModel.php`, ajoutez ce qui suit :

```php
<?php

namespace App\Models;

use CodeIgniter\Model;
use Exception;

class ClientModel extends Model
{
    protected $table = 'client';
    protected $allowedFields = [
        'name',
        'email',
        'retainer_fee'
    ];
    protected $updatedField = 'updated_at';

    public function findClientById($id)
    {
        $client = $this
            ->asArray()
            ->where(['id' => $id])
            ->first();

        if (!$client) throw new Exception('Could not find client for specified ID');

        return $client;
    }
}
```

Le champ `$table` permet au modèle de savoir avec quelle table de base de données il fonctionne principalement. `$allowedFields` permet au modèle de savoir quelles colonnes de la table peuvent être mises à jour. La fonction `findClientById` fournit une abstraction propre pour extraire un client de la base de données en fonction de l'`id` fourni.

Une fois les modèles et la base de données implémentés, les utilisateurs peuvent être ajoutés et authentifiés. Les utilisateurs autorisés peuvent également interagir avec la clientèle actuelle.

### Implémentation de JWT

Les [Web JSON Web Tokens](https://jwt.io/) seront utilisés pour authentifier les utilisateurs et empêcher les utilisateurs non autorisés d'afficher la liste des clients. Pour que cela fonctionne, l'API fournit un token lorsque l'utilisateur s'inscrit ou se connecte correctement. Ce token sera ajouté à l'en-tête des requêtes suivantes pour s'assurer que l'API peut identifier l'utilisateur à l'origine de la requête. Dans ce tutoriel, l'ensemble [firebase/php-jwt](https://github.com/ParitoshVaidya/CodeIgniter-JWT-Sample) sera utilisé pour générer les tokens. Exécutez ce qui suit pour l'installer à l'aide de Composer :

    composer require firebase/php-jwt

Une fois l'installation terminée, ajoutez les éléments suivants à votre fichier `.env` :


```
#JWT_SECRET_KEY key is the secret key used by the application to sign JWTS. Pick a stronger one for production.
JWT_SECRET_KEY=mIKm1jDusb9Z7BfImUuBIcEfpso4yVV5TWtb3Jxf
#JWT_TIME_TO_LIVE indicates the validity period of a signed JWT (in milliseconds)
JWT_TIME_TO_LIVE=3600
```

Ensuite, créez une fonction d'aide pour obtenir la clé secrète dans la classe Services. Accédez à *App/Config/Services.php* (`app/Config/Services.php`) et ajoutez ce qui suit :

```php
<?php

namespace Config;

use CodeIgniter\Config\BaseService;

/**
 * Services Configuration file.
 *
 * Services are simply other classes/libraries that the system uses
 * to do its job. This is used by CodeIgniter to allow the core of the
 * framework to be swapped out easily without affecting the usage within
 * the rest of your application.
 *
 * This file holds any application-specific services, or service overrides
 * that you might need. An example has been included with the general
 * method format you should use for your service methods. For more examples,
 * see the core Services file at system/Config/Services.php.
 */
class Services extends BaseService
{
    /*
     * public static function example($getShared = true)
     * {
     *     if ($getShared) {
     *         return static::getSharedInstance('example');
     *     }
     *
     *     return new \CodeIgniter\Example();
     * }
     */
	public static function getSecretKey(){
	    return getenv('JWT_SECRET_KEY');
	} 

}
```

### Création d'un JWT Helper

Pour faciliter la génération et la vérification des tokens, un [fichier Helper](https://codeigniter.com/user_guide/general/helpers.html) sera créé. Cela nous permet de séparer les préoccupations dans notre application. Dans le répertoire *App/Helpers* (`app/Helpers/`), créez un fichier nommé `jwt_helper.php`. Votre fichier devrait ressembler à ceci :

app/Helpers/jwt_helper.php

```php
<?php

use App\Models\UserModel;
use Config\Services;
use Firebase\JWT\JWT;

function getJWTFromRequest($authenticationHeader): string
{
    if (is_null($authenticationHeader)) { //JWT is absent
        throw new Exception('Missing or invalid JWT in request');
    }
    //JWT is sent from client in the format Bearer XXXXXXXXX
    return explode(' ', $authenticationHeader)[1];
}

function validateJWTFromRequest(string $encodedToken)
{
    $key = Services::getSecretKey();
    $decodedToken = JWT::decode($encodedToken, $key, ['HS256']);
    $userModel = new UserModel();
    $userModel->findUserByEmailAddress($decodedToken->email);
}

function getSignedJWTForUser(string $email)
{
    $issuedAtTime = time();
    $tokenTimeToLive = getenv('JWT_TIME_TO_LIVE');
    $tokenExpiration = $issuedAtTime + $tokenTimeToLive;
    $payload = [
        'email' => $email,
        'iat' => $issuedAtTime,
        'exp' => $tokenExpiration,
    ];

    $jwt = JWT::encode($payload, Services::getSecretKey());
    return $jwt;
}
```

La fonction `getJWTFromRequest` vérifie l'en-tête [Authorization](https://stackoverflow.com/questions/5110361/what-is-the-http-authorization-environment-variable) de la requête entrante et renvoie la valeur du token. Si l'en-tête est manquant, une exception est déclenchée. Celle-ci entraîne à son tour le renvoi d'une réponse `HTTP_UNAUTHORIZED` (401).

La fonction validateJWTFromRequest utilise le token obtenu au moyen de la fonction `getJWTFromRequest`. Elle décode ce token afin d'obtenir l'e-mail pour lequel la clé a été générée. Elle tente ensuite de trouver un utilisateur avec cette adresse e-mail dans la base de données. Si l'utilisateur n'a pas été trouvé, le modèle d'utilisateur déclenche une exception qui est détectée et renvoyée à l'utilisateur sous la forme d'une réponse `HTTP_UNAUTHORIZED` (401).

La fonction `getSignedJWTForUser` est utilisée pour générer un token pour un utilisateur authentifié. Le JWT codé contient les détails suivants :

*    Adresse e-mail de l'utilisateur authentifié. Elle est utilisée dans les requêtes ultérieures pour valider la source de la requête.
*    Heure à laquelle le token a été généré (iat).
*    Heure d'expiration du token (exp). Elle est obtenue en ajoutant la valeur JWT_TIME_TO_LIVE de notre fichier .env à l'heure actuelle.


### Création filtre d'authentification

Dans le répertoire *App/Filters* (`app/Filters/`), créez un fichier nommé `JWTAuthenticationFilter.php`. Ce filtre permet de vérifier l'API du JWT avant de transmettre la requête au contrôleur. Si aucun JWT n'est fourni ou si le JWT fourni a expiré, une réponse HTTP_UNAUTHORIZED (401) est renvoyée avec un message d'erreur approprié. Ajoutez ce qui suit à votre fichier :

app/Filters/JWTAuthenticationFilter.php

```php
<?php

namespace App\Filters;

use CodeIgniter\API\ResponseTrait;
use CodeIgniter\Filters\FilterInterface;
use CodeIgniter\HTTP\RequestInterface;
use CodeIgniter\HTTP\ResponseInterface;
use Config\Services;
use Exception;

class JWTAuthenticationFilter implements FilterInterface
{
    use ResponseTrait;

    public function before(RequestInterface $request, $arguments = null)
    {
        $authenticationHeader = $request->getServer('HTTP_AUTHORIZATION');

        try {

            helper('jwt');
            $encodedToken = getJWTFromRequest($authenticationHeader);
            validateJWTFromRequest($encodedToken);
            return $request;

        } catch (Exception $e) {

            return Services::response()
                ->setJSON(
                    [
                        'error' => $e->getMessage()
                    ]
                )
                ->setStatusCode(ResponseInterface::HTTP_UNAUTHORIZED);

        }
    }

    public function after(RequestInterface $request,
                          ResponseInterface $response,
                          $arguments = null)
    {
    }
}
```

Comme vous pouvez le voir, JWT Helper est chargé en premier, puis les fonctions `getJWTFromRequest` et `validateJWTFromRequest` sont utilisées pour s'assurer que la requête provient d'un utilisateur authentifié avec un token valide.

Enregistrez votre filtre JWTAuthentication et spécifiez le chemin à protéger. Cette opération s'effectue dans le fichier *App/Config/Filters.php*. Mettez à jour les tableaux `$aliases` et `$filters` comme suit :

```php
<?php

namespace Config;

use CodeIgniter\Config\BaseConfig;
use CodeIgniter\Filters\CSRF;
use CodeIgniter\Filters\DebugToolbar;
use CodeIgniter\Filters\Honeypot;

class Filters extends BaseConfig
{
    /**
     * Configures aliases for Filter classes to
     * make reading things nicer and simpler.
     *
     * @var array
     */
    public $aliases = [
        'csrf'     => CSRF::class,
        'toolbar'  => DebugToolbar::class,
        'honeypot' => Honeypot::class,
	'auth' => JWTAuthenticationFilter::class
    ];

    /**
     * List of filter aliases that are always
     * applied before and after every request.
     *
     * @var array
     */
    public $globals = [
        'before' => [
            // 'honeypot',
            // 'csrf',
        ],
        'after' => [
            // 'toolbar',
            // 'honeypot',
        ],
    ];

    /**
     * List of filter aliases that works on a
     * particular HTTP method (GET, POST, etc.).
     *
     * Example:
     * 'post' => ['csrf', 'throttle']
     *
     * @var array
     */
    public $methods = [];

    /**
     * List of filter aliases that should run on any
     * before or after URI patterns.
     *
     * Example:
     * 'isLoggedIn' => ['before' => ['account/*', 'profiles/*']]
     *
     * @var array
     */
    public $filters = [
      'auth' => [
        'before' => [
            'client/*',
            'client'
      ],
    ]
  ];
}
```

Version simplifiée

```php
<?php 
namespace Config;

use App\Filters\JWTAuthenticationFilter;
use CodeIgniter\Config\BaseConfig;

class Filters extends BaseConfig
{
    public $aliases = [
        'csrf' => CSRF::class,
        'toolbar' => DebugToolbar::class,
        'honeypot' => \CodeIgniter\Filters\Honeypot::class,
        'auth' => JWTAuthenticationFilter::class // add this line
    ];

    // global filters
    // method filters
    public $filters = [
      'auth' => [
        'before' => [
            'client/*',
            'client'
      ],
    ]
  ];
}
```

>REMARQUE : la barre d'outils de débogage est préchargée par défaut. Il existe des conflits connus, car la barre d'outils de débogage est [toujours en construction](https://codeigniter4.github.io/CodeIgniter4/testing/debugging.html#the-debug-toolbar). Pour la désactiver, ajoutez un commentaire sur l'élément 'toolbar' dans le tableau $globals.

En ajoutant ces éléments, la fonction `before` dans *JWTAuthenticationFilter.php* est appelée chaque fois qu'une requête est envoyée à un endpoint commençant par le **client**. Cela signifie que le contrôleur reçoit/traite la requête uniquement si son en-tête contient un token valide.

Même si nous n'avons pas de contrôleur, nous pouvons vérifier que notre application fonctionne jusqu'à présent.  

Lancer le serveur

    cd ~/ci-secure-api
    php spark serve

```
CodeIgniter development server started on http://localhost:8080
Press Control-C to stop.
[Mon Dec 27 16:07:12 2021] PHP 8.0.14 Development Server (http://localhost:8080) started
```

Ouvrez Postman et faites une requête GET à http://localhost:8080/client.
Par curl 

    curl localhost:8080/client

```
{
    "error": "Missing or invalid JWT in request"
}
```


Ouvrez ensuite le fichier *App/Controllers/BaseController.php*

`BaseController` étend le `Controller` de CodeIgniter, qui fournit des assistants et d'autres fonctions facilitant le traitement des requêtes entrantes. L'une de ces fonctions est validate qui utilise le service de validation de CodeIgniter pour vérifier une requête par rapport aux règles (et aux messages d'erreur si nécessaire) spécifiées dans les fonctions de notre contrôleur. Cette fonction donne de bons résultats avec les requêtes de formulaire (form-data avec Postman). Cependant, elle ne serait pas en mesure de valider les requêtes JSON brutes envoyées à notre API. En effet, le contenu de la requête JSON est stocké dans le champ body tandis que le contenu de la requête form-data est stocké dans le champ post.

Pour contourner ce problème, nous allons écrire une fonction qui vérifie les deux champs dans une requête afin d'obtenir son contenu.

Ensuite, déclarez une fonction qui exécute le service de validation par rapport à `$input` de notre fonction précédente. Cette fonction est presque la même que la fonction `validate` intégrée, sauf qu'au lieu d'exécuter la vérification sur IncomingRequest, nous l'exécutons sur l'entrée que nous avons capturée à partir de la fonction `getRequestInput`.


Le fichier final avec les modifications `app/Controllers/BaseController.php` 

```php
<?php
namespace App\Controllers;

/**
 * Class BaseController
 *
 * BaseController provides a convenient place for loading components
 * and performing functions that are needed by all your controllers.
 * Extend this class in any new controllers:
 *     class Home extends BaseController
 *
 * For security be sure to declare any new methods as protected or private.
 *
 * @package CodeIgniter
 */

use CodeIgniter\Controller;
use CodeIgniter\HTTP\ResponseInterface;
use CodeIgniter\HTTP\IncomingRequest;
use CodeIgniter\Validation\Exceptions\ValidationException;
use Config\Services;

class BaseController extends Controller
{

	/**
	 * An array of helpers to be loaded automatically upon
	 * class instantiation. These helpers will be available
	 * to all other controllers that extend BaseController.
	 *
	 * @var array
	 */
	protected $helpers = [];

	/**
	 * Constructor.
	 */
	public function initController(\CodeIgniter\HTTP\RequestInterface $request, \CodeIgniter\HTTP\ResponseInterface $response, \Psr\Log\LoggerInterface $logger)
	{
		// Do Not Edit This Line
		parent::initController($request, $response, $logger);

		//--------------------------------------------------------------------
		// Preload any models, libraries, etc, here.
		//--------------------------------------------------------------------
		// E.g.:
		// $this->session = \Config\Services::session();
	}

    public function getResponse(array $responseBody,
                                int $code = ResponseInterface::HTTP_OK)
    {
        return $this
            ->response
            ->setStatusCode($code)
            ->setJSON($responseBody);
    }

    public function getRequestInput(IncomingRequest $request){
        $input = $request->getPost();
        if (empty($input)) {
            //convert request body to associative array
            $input = json_decode($request->getBody(), true);
        }
        return $input;
    }

    public function validateRequest($input, array $rules, array $messages = []){
        $this->validator = Services::Validation()->setRules($rules);
        // If you replace the $rules array with the name of the group
        if (is_string($rules)) {
            $validation = config('Validation');

            // If the rule wasn't found in the \Config\Validation, we
            // should throw an exception so the developer can find it.
            if (!isset($validation->$rules)) {
                throw ValidationException::forRuleNotFound($rules);
            }

            // If no error message is defined, use the error message in the Config\Validation file
            if (!$messages) {
                $errorName = $rules . '_errors';
                $messages = $validation->$errorName ?? [];
            }

            $rules = $validation->$rules;
        }
        return $this->validator->setRules($rules, $messages)->run($input);
    }


}
```

Une fois cela mis en place, ajoutons la logique pour enregistrer et authentifier les utilisateurs.


Contrôleur d'authentification

Ensuite, créez un fichier nommé *Auth.php* dans le répertoire `App/Controllers`. Mettez à jour le fichier comme indiqué ci-dessous :

```php
<?php
namespace App\Controllers;

use App\Models\UserModel;
use CodeIgniter\HTTP\Response;
use CodeIgniter\HTTP\ResponseInterface;
use Exception;
use ReflectionException;

class Auth extends BaseController
{
    /**
     * Register a new user
     * @return Response
     * @throws ReflectionException
     */
    public function register()
    {
        $rules = [
            'name' => 'required',
            'email' => 'required|min_length[6]|max_length[50]|valid_email|is_unique[user.email]',
            'password' => 'required|min_length[8]|max_length[255]'
        ];

        $input = $this->getRequestInput($this->request);

        if (!$this->validateRequest($input, $rules)) {
            return $this
                ->getResponse(
                    $this->validator->getErrors(),
                    ResponseInterface::HTTP_BAD_REQUEST
                );
        }

        $userModel = new UserModel();
        $userModel->save($input);

        return $this
            ->getJWTForUser(
                $input['email'],
                ResponseInterface::HTTP_CREATED
            );
    }

    /**
     * Authenticate Existing User
     * @return Response
     */
    public function login()
    {
        $rules = [
            'email' => 'required|min_length[6]|max_length[50]|valid_email',
            'password' => 'required|min_length[8]|max_length[255]|validateUser[email, password]'
        ];

        $errors = [
            'password' => [
                'validateUser' => 'Invalid login credentials provided'
            ]
        ];

        $input = $this->getRequestInput($this->request);

        if (!$this->validateRequest($input, $rules, $errors)) {
            return $this
                ->getResponse(
                    $this->validator->getErrors(),
                    ResponseInterface::HTTP_BAD_REQUEST
                );
        }

        return $this->getJWTForUser($input['email']);
    }

    private function getJWTForUser(
        string $emailAddress,
        int $responseCode = ResponseInterface::HTTP_OK
    )
    {
        try {
            $model = new UserModel();
            $user = $model->findUserByEmailAddress($emailAddress);
            unset($user['password']);

            helper('jwt');

            return $this
                ->getResponse(
                    [
                        'message' => 'User authenticated successfully',
                        'user' => $user,
                        'access_token' => getSignedJWTForUser($emailAddress)
                    ]
                );
        } catch (Exception $exception) {
            return $this
                ->getResponse(
                    [
                        'error' => $exception->getMessage(),
                    ],
                    $responseCode
                );
        }
    }
}
```

### Enregistrement

Pour enregistrer un nouvel utilisateur avec succès, les champs suivants sont obligatoires :

*    Un nom.
*    Une adresse e-mail dans un format valide ne comportant pas moins de 8 caractères et pas plus de 255 caractères.
*    Un mot de passe de 8 caractères minimum et de 255 caractères maximum.

La requête entrante est vérifiée par rapport aux règles spécifiées. Les requêtes non valides sont ignorées avec un code (400) `HTTP_BAD_REQUEST` et un message d'erreur. Si la requête est valide, les données utilisateur sont enregistrées et un token est renvoyé avec les détails enregistrés de l'utilisateur (à l'exception du mot de passe). La réponse `HTTP_CREATED` (201) informe le client qu'une nouvelle ressource a été créée.

En effectuant une requête POST sur le endpoint du registre (http://localhost:8080/auth/register) avec un nom (name), une adresse e-mail (e-mail) et un mot de passe (password) valides, vous obtenez une réponse similaire à celle présentée ci-dessous :

    curl -d "name=Gaspar Beauchesne&email=GasparBeauchesne@jourrapide.com&password=b25GraW8NL5s" http://localhost:8080/auth/register

```
{
    "message": "User authenticated successfully",
    "user": {
        "id": "1",
        "name": "Gaspar Beauchesne",
        "email": "GasparBeauchesne@jourrapide.com",
        "updated_at": null,
        "created_at": "2021-12-27 17:51:09"
    },
    "access_token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJlbWFpbCI6Ikdhc3BhckJlYXVjaGVzbmVAam91cnJhcGlkZS5jb20iLCJpYXQiOjE2NDA2Mjc0NjksImV4cCI6MTY0MDYzMTA2OX0.cp8Ry2RXQZAhUCw5OTli4KyoYY61CD_pmkNVyzGzeGI"
}
```

### Authentification

Une authentification réussie nécessite les éléments suivants :

*    Une adresse e-mail dans un format valide ne comportant pas moins de 8 caractères et pas plus de 255 caractères. En outre, l'adresse e-mail doit correspondre à celle d'un utilisateur enregistré.
*    Un mot de passe de 8 caractères minimum et de 255 caractères maximum. Comme pour l'adresse e-mail, le hachage du mot de passe fourni doit correspondre au hachage du mot de passe stocké associé à l'adresse e-mail fournie.

Cependant, faire de même pour le endpoint de connexion (http://localhost:8080/auth/login) entraînerait une erreur de serveur interne (code HTTP 500). La raison est que nous utilisons une fonction `validateUser` dans nos règles de validation que nous n'avons pas encore créées.

### Validation de l'utilisateur

Créez un nouveau répertoire appelé *Validation* dans le répertoire `app`. Dans le dossier app/Validation, créez un fichier nommé *UserRules.php* `app/Validation/UserRules.php` et ajoutez le code suivant au fichier :

```php
<?php

namespace App\Validation;

use App\Models\UserModel;
use Exception;

class UserRules
{
    public function validateUser(string $str, string $fields, array $data): bool
    {
        try {
            $model = new UserModel();
            $user = $model->findUserByEmailAddress($data['email']);
            return password_verify($data['password'], $user['password']);
        } catch (Exception $e) {
            return false;
        }
    }
}
```

Ouvrez ensuite le fichier *App/Config/Validation.php* et modifiez le tableau `$ruleSets` pour inclure vos `UserRules` 

```php
<?php namespace Config;

class Validation
{
	//--------------------------------------------------------------------
	// Setup
	//--------------------------------------------------------------------

	/**
	 * Stores the classes that contain the
	 * rules that are available.
	 *
	 * @var array
	 */
	public $ruleSets = [
		\CodeIgniter\Validation\Rules::class,
		\CodeIgniter\Validation\FormatRules::class,
		\CodeIgniter\Validation\FileRules::class,
		\CodeIgniter\Validation\CreditCardRules::class,
        \App\Validation\UserRules::class,
    ];

	/**
	 * Specifies the views that are used to display the
	 * errors.
	 *
	 * @var array
	 */
	public $templates = [
		'list'   => 'CodeIgniter\Validation\Views\list',
		'single' => 'CodeIgniter\Validation\Views\single',
	];

	//--------------------------------------------------------------------
	// Rules
	//--------------------------------------------------------------------
}
```

Avec les règles de validation personnalisées mises en place, la requête d'authentification fonctionne comme prévu. Testez ceci en envoyant une requête POST HTTP au endpoint http://localhost:8080/auth/login avec les détails de l'utilisateur créé précédemment :

    curl -d "name=Gaspar Beauchesne&email=GasparBeauchesne@jourrapide.com&password=b25GraW8NL5s" http://localhost:8080/auth/login

```
{
    "message": "User authenticated successfully",
    "user": {
        "id": "1",
        "name": "Gaspar Beauchesne",
        "email": "GasparBeauchesne@jourrapide.com",
        "updated_at": null,
        "created_at": "2021-12-27 17:51:09"
    },
    "access_token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJlbWFpbCI6Ikdhc3BhckJlYXVjaGVzbmVAam91cnJhcGlkZS5jb20iLCJpYXQiOjE2NDA2MzA0OTAsImV4cCI6MTY0MDYzNDA5MH0.ykLLGfTlYJgpvsZd9uhvPdKkesqk-859po5J_RLXsAA"
}
```

### Création d'un contrôleur client

Pour le contrôleur client, nous allons spécifier les chemins dans le fichier *app/Config/Routes.php*. Ouvrez le fichier et ajoutez-y les chemins

```php
<?php namespace Config;

// Create a new instance of our RouteCollection class.
$routes = Services::routes();

// Load the system's routing file first, so that the app and ENVIRONMENT
// can override as needed.
if (file_exists(SYSTEMPATH . 'Config/Routes.php'))
{
	require SYSTEMPATH . 'Config/Routes.php';
}

/**
 * --------------------------------------------------------------------
 * Router Setup
 * --------------------------------------------------------------------
 */
$routes->setDefaultNamespace('App\Controllers');
$routes->setDefaultController('Home');
$routes->setDefaultMethod('index');
$routes->setTranslateURIDashes(false);
$routes->set404Override();
$routes->setAutoRoute(true);

/**
 * --------------------------------------------------------------------
 * Route Definitions
 * --------------------------------------------------------------------
 */

// We get a performance increase by specifying the default
// route since we don't have to scan directories.
$routes->get('/', 'Home::index');

/**
 * --------------------------------------------------------------------
 * Additional Routing
 * --------------------------------------------------------------------
 *
 * There will often be times that you need additional routing and you
 * need it to be able to override any defaults in this file. Environment
 * based routes is one such time. require() additional route files here
 * to make that happen.
 *
 * You will have access to the $routes object within that file without
 * needing to reload it.
 */
if (file_exists(APPPATH . 'Config/' . ENVIRONMENT . '/Routes.php'))
{
	require APPPATH . 'Config/' . ENVIRONMENT . '/Routes.php';
}

$routes->get('client', 'Client::index');
$routes->post('client', 'Client::store');
$routes->get('client/(:num)', 'Client::show/$1');
$routes->post('client/(:num)', 'Client::update/$1');
$routes->delete('client/(:num)', 'Client::destroy/$1');
```

En procédant ainsi, votre API est capable de traiter les requêtes avec le même endpoint, mais des verbes HTTP différents.

Ensuite, dans le répertoire *App/Controllers*, créez un fichier appelé `Client.php`. Le contenu du fichier doit être le suivant :

```php
<?php

namespace App\Controllers;

use App\Models\ClientModel;
use CodeIgniter\HTTP\Response;
use CodeIgniter\HTTP\ResponseInterface;
use Exception;

class Client extends BaseController
{
    /**
     * Get all Clients
     * @return Response
     */
    public function index()
    {
        $model = new ClientModel();
        return $this->getResponse(
            [
                'message' => 'Clients retrieved successfully',
                'clients' => $model->findAll()
            ]
        );
    }

    /**
     * Create a new Client
     */
    public function store()
    {
        $rules = [
            'name' => 'required',
            'email' => 'required|min_length[6]|max_length[50]|valid_email|is_unique[client.email]',
            'retainer_fee' => 'required|max_length[255]'
        ];

        $input = $this->getRequestInput($this->request);

        if (!$this->validateRequest($input, $rules)) {
            return $this
                ->getResponse(
                    $this->validator->getErrors(),
                    ResponseInterface::HTTP_BAD_REQUEST
                );
        }

        $clientEmail = $input['email'];

        $model = new ClientModel();
        $model->save($input);

        $client = $model->where('email', $clientEmail)->first();

        return $this->getResponse(
            [
                'message' => 'Client added successfully',
                'client' => $client
            ]
        );
    }

    /**
     * Get a single client by ID
     */
    public function show($id)
    {
        try {

            $model = new ClientModel();
            $client = $model->findClientById($id);

            return $this->getResponse(
                [
                    'message' => 'Client retrieved successfully',
                    'client' => $client
                ]
            );

        } catch (Exception $e) {
            return $this->getResponse(
                [
                    'message' => 'Could not find client for specified ID'
                ],
                ResponseInterface::HTTP_NOT_FOUND
            );
        }
    }

    public function update($id)
    {
        try {

            $model = new ClientModel();
            $model->findClientById($id);

            $input = $this->getRequestInput($this->request);

            $model->update($id, $input);
            $client = $model->findClientById($id);

            return $this->getResponse(
                [
                    'message' => 'Client updated successfully',
                    'client' => $client
                ]
            );

        } catch (Exception $exception) {

            return $this->getResponse(
                [
                    'message' => $exception->getMessage()
                ],
                ResponseInterface::HTTP_NOT_FOUND
            );
        }
    }

    public function destroy($id)
    {
        try {

            $model = new ClientModel();
            $client = $model->findClientById($id);
            $model->delete($client);

            return $this
                ->getResponse(
                    [
                        'message' => 'Client deleted successfully',
                    ]
                );

        } catch (Exception $exception) {
            return $this->getResponse(
                [
                    'message' => $exception->getMessage()
                ],
                ResponseInterface::HTTP_NOT_FOUND
            );
        }
    }
}
```



Les fonctions `index`, `store `et `show` sont utilisées pour traiter les requêtes d'affichage de tous les clients, d'ajout d'un nouveau client et d'affichage d'un seul client respectivement.

Ensuite, créez deux fonctions `update` et `destroy`. La fonction `update` sera utilisée pour traiter les requêtes de modification d'un client. Aucun des champs n'est requis, donc toute valeur attendue qui n'est pas fournie dans la requête est supprimée avant la mise à jour du client dans la base de données. La fonction `destroy` traite les requêtes de suppression d'un client particulier.


## curl

* [How to test a REST api from command line with curl](https://www.codepedia.org/ama/how-to-test-a-rest-api-from-command-line-with-curl/)
* [Comment cURL POST à ​​partir de la ligne de commande](https://fre.applersg.com/how-curl-post-from-command-line)
* [Comment utiliser la commande Curl sous Linux ?](https://www.hostinger.fr/tutoriels/comment-utiliser-la-commande-curl-sous-linux)
* [cURL: Add Header, Multiple Headers, Authorization](https://www.shellhacks.com/curl-add-header-multiple-headers-authorization/)
* [Test a REST API with curl](https://www.baeldung.com/curl-rest)

## API ready

### Lancer le serveur

Maintenant que ces éléments sont en place, notre API est prête à être consommée. Redémarrez votre application et testez-la en envoyant des requêtes (via Postman, cURL ou votre application préférée).

    php spark serve

On se connecte avec l'utilisateur enregistré

    curl -d "name=Gaspar Beauchesne&email=GasparBeauchesne@jourrapide.com&password=b25GraW8NL5s" http://localhost:8080/auth/login

```json
{
    "message": "User authenticated successfully",
    "user": {
        "id": "1",
        "name": "Gaspar Beauchesne",
        "email": "GasparBeauchesne@jourrapide.com",
        "updated_at": null,
        "created_at": "2021-12-27 17:51:09"
    },
    "access_token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJlbWFpbCI6Ikdhc3BhckJlYXVjaGVzbmVAam91cnJhcGlkZS5jb20iLCJpYXQiOjE2NDA4NjE1OTYsImV4cCI6MTY0MDg2NTE5Nn0.Z4SK0hlOaAioDMxHYw3LevlF7olG9IzXuImlvk3-KDQ"
```

### Ajout d'un token d'accès

Une fois le processus d'enregistrement et de connexion terminé, copiez la valeur de `access_token` de la réponse. Ensuite, cliquez sur l'onglet `Authorization`, sélectionnez `Bearer token` dans la liste déroulante et collez la valeur de `access_token` copiée précédemment :

    "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJlbWFpbCI6Ikdhc3BhckJlYXVjaGVzbmVAam91cnJhcGlkZS5jb20iLCJpYXQiOjE2NDA4NjE1OTYsImV4cCI6MTY0MDg2NTE5Nn0.Z4SK0hlOaAioDMxHYw3LevlF7olG9IzXuImlvk3-KDQ"

### Création d'un nouveau client

Pour créer un nouveau client, envoyez une requête HTTP POST à http://localhost:8080/client avec la clé d'accès de l'utilisateur

```bash
curl -i -X POST "http://localhost:8080/client" \
-H "accept: */*" -H "Authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJlbWFpbCI6Ikdhc3BhckJlYXVjaGVzbmVAam91cnJhcGlkZS5jb20iLCJpYXQiOjE2NDA4NjE1OTYsImV4cCI6MTY0MDg2NTE5Nn0.Z4SK0hlOaAioDMxHYw3LevlF7olG9IzXuImlvk3-KDQ"  \
-H "Content-Type: application/json" -d "{\"name\":\"Christian David\",\"email\":\"ChristianDavid@jourrapide.com\",\"password\":\"Zo7hifai8B\",\"retainer_fee\":\"15000\"}"
```

Résultat

```json
HTTP/1.1 200 OK
Host: localhost:8080
Date: Thu, 30 Dec 2021 10:56:48 GMT
Connection: close
X-Powered-By: PHP/8.0.14
Cache-control: no-store, max-age=0, no-cache
Content-Type: application/json; charset=UTF-8
Debugbar-Time: 1640861808
Debugbar-Link: http://localhost:8080/index.php?debugbar_time=1640861808

{
    "message": "Client added successfully",
    "client": {
        "id": "12",
        "name": "Christian David",
        "email": "ChristianDavid@jourrapide.com",
        "retainer_fee": "15000",
        "updated_at": null,
        "created_at": "2021-12-30 10:56:48"
    }
}
```

### Récupération de la liste de tous les clients

Pour récupérer la liste des clients créés jusqu'à présent, envoyez une requête HTTP GET à http://localhost:8080/client

```bash
curl -s -X GET "http://localhost:8080/client" \
-H "accept: application/json" \
-H "Authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJlbWFpbCI6Ikdhc3BhckJlYXVjaGVzbmVAam91cnJhcGlkZS5jb20iLCJpYXQiOjE2NDA4NjE1OTYsImV4cCI6MTY0MDg2NTE5Nn0.Z4SK0hlOaAioDMxHYw3LevlF7olG9IzXuImlvk3-KDQ" | jq
```

Résultat

```json
{
    "message": "Clients retrieved successfully",
    "clients": [
        {
            "id": "1",
            "name": "Colt Hintz",
            "email": "mackenzie72@hotmail.com",
            "retainer_fee": "21526147",
            "updated_at": null,
            "created_at": "2021-12-27 14:44:06"
        },
        {
            "id": "2",
            "name": "Ms. Linda Romaguera DVM",
            "email": "frederick.herzog@hotmail.com",
            "retainer_fee": "39778512",
            "updated_at": null,
            "created_at": "2021-12-27 14:44:06"
        },
        {
            "id": "3",
            "name": "Juston Effertz",
            "email": "ansley07@sporer.biz",
            "retainer_fee": "81860215",
            "updated_at": null,
            "created_at": "2021-12-27 14:44:06"
        },
        {
            "id": "4",
            "name": "Mrs. Melissa Auer",
            "email": "maritza14@hotmail.com",
            "retainer_fee": "94636348",
            "updated_at": null,
            "created_at": "2021-12-27 14:44:06"
        },
        {
            "id": "5",
            "name": "Giles Predovic",
            "email": "mkunze@gleason.biz",
            "retainer_fee": "1522348",
            "updated_at": null,
            "created_at": "2021-12-27 14:44:06"
        },
        {
            "id": "6",
            "name": "Henderson Keeling",
            "email": "brannon23@sipes.org",
            "retainer_fee": "15540299",
            "updated_at": null,
            "created_at": "2021-12-27 14:44:06"
        },
        {
            "id": "7",
            "name": "Walton Hettinger",
            "email": "abarrows@kautzer.info",
            "retainer_fee": "70656137",
            "updated_at": null,
            "created_at": "2021-12-27 14:44:06"
        },
        {
            "id": "8",
            "name": "Cydney Bernier IV",
            "email": "elisa.dickens@torphy.com",
            "retainer_fee": "53441851",
            "updated_at": null,
            "created_at": "2021-12-27 14:44:06"
        },
        {
            "id": "9",
            "name": "Dr. Kathleen Greenfelder",
            "email": "iwolf@boehm.com",
            "retainer_fee": "11875923",
            "updated_at": null,
            "created_at": "2021-12-27 14:44:06"
        },
        {
            "id": "10",
            "name": "Fannie Turner",
            "email": "ralph43@yahoo.com",
            "retainer_fee": "60052006",
            "updated_at": null,
            "created_at": "2021-12-27 14:44:06"
        },
        {
            "id": "11",
            "name": "Saber Doyon",
            "email": "SaberDoyon@jourrapide.com",
            "retainer_fee": "10000",
            "updated_at": null,
            "created_at": "2021-12-28 11:07:55"
        },
        {
            "id": "12",
            "name": "Christian David",
            "email": "ChristianDavid@jourrapide.com",
            "retainer_fee": "15000",
            "updated_at": null,
            "created_at": "2021-12-30 10:56:48"
        }
    ]
}
```

### Client par id

```bash
curl -s -X GET "http://localhost:8080/client/11" \
-H "accept: application/json" \
-H "Authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJlbWFpbCI6Ikdhc3BhckJlYXVjaGVzbmVAam91cnJhcGlkZS5jb20iLCJpYXQiOjE2NDA4NjY3MTgsImV4cCI6MTY0MDg3MDMxOH0.unt-ayxz3UhcvuKp4aGDkT1kfTx8m47CFC8TBG178iM" 
```

Résultat

```json
{
    "message": "Client retrieved successfully",
    "client": {
        "id": "11",
        "name": "Saber Doyon",
        "email": "SaberDoyon@jourrapide.com",
        "retainer_fee": "10000",
        "updated_at": null,
        "created_at": "2021-12-28 11:07:55"
    }
}
```

### Mise à jour d'un client existant

Le champ name est requis

```bash
$curl -i -X POST "http://localhost:8080/client/12" \
-H "accept: application/json" \
-H "Authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJlbWFpbCI6Ikdhc3BhckJlYXVjaGVzbmVAam91cnJhcGlkZS5jb20iLCJpYXQiOjE2NDA4ODAxNTksImV4cCI6MTY0MDg4Mzc1OX0.6eDL8j1NcekqCkVBGYn12phcQQTqlFzkkOMwQG69AwE" \
-H "Content-Type: application/json" -d "{\"name\":\"Christian David\",\"retainer_fee\":\"55000\"}"
```

Résultat

```
HTTP/1.1 200 OK
Host: localhost:8080
Date: Thu, 30 Dec 2021 16:10:22 GMT
Connection: close
X-Powered-By: PHP/8.0.14
Cache-control: no-store, max-age=0, no-cache
Content-Type: application/json; charset=UTF-8
Debugbar-Time: 1640880622
Debugbar-Link: http://localhost:8080/index.php?debugbar_time=1640880622

{
    "message": "Client updated successfully",
    "client": {
        "id": "12",
        "name": "Christian David",
        "email": "ChristianDavid@jourrapide.com",
        "retainer_fee": "55000",
        "updated_at": null,
        "created_at": "2021-12-30 10:56:48"
    }
}
```

### Suppression d'un client existant

Supprimer client id=11

```bash
curl -s -X DELETE "http://localhost:8080/client/11"  \
-H "accept: application/json" \
-H "Authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJlbWFpbCI6Ikdhc3BhckJlYXVjaGVzbmVAam91cnJhcGlkZS5jb20iLCJpYXQiOjE2NDA4NzU0NzQsImV4cCI6MTY0MDg3OTA3NH0.q0BMHi4x2WPPGJ5ZaeiiucdRrW6ECgTCDI7fszt0_Wo"
```

Résultat

```json 
{
    "message": "Client deleted successfully"
}
```

### Conclusion

Dans cet article, nous avons créé une API PHP à l'aide de CodeIgniter. Cela a permis l'exécution d'opérations CRUD (Create [Créer], Read [Lire], Update [Mettre à jour], Delete [Supprimer]) de base sur une ressource (client). En outre, nous avons ajouté une couche de sécurité en limitant l'accès à la ressource. Nous avons également appris à structurer notre projet de manière à séparer les concerns et à rendre notre application plus faiblement couplée.

## Codeigniter Nginx server configuration 

* [CodeIgniter](https://unit.nginx.org/howto/codeigniter/)
* [Nginx configuration for CodeIgniter](https://wylu.me/posts/ec244fdf/) 


### Web Server Site Configuration

#### Recommended Nginx Configuration

To use Nginx, you should install PHP as an FPM SAPI. You may use the following Nginx configuration, replacing `root` value with the actual path for Codeignitor porject and `server_name` with the actual hostname to serve.

```nginx
server {
        server_name domain.tld;

        root /var/www/codeignitor;
        index index.html index.php;

        # set expiration of assets to MAX for caching
        #location ~* \.(ico|css|js|gif|jpe?g|png)(\?[0-9]+)?$ {
        #        expires max;
        #        log_not_found off;
        #}

        location / {
                # Check if a file or directory index file exists, else route it to index.php.
                try_files $uri $uri/ /index.php;
        }

        location ~* \.php$ {
                fastcgi_pass 127.0.0.1:9000;
		fastcgi_pass unix:/var/run/php5-fpm.sock;
                include fastcgi.conf;
		#fastcgi_param CI_ENV 'production';
        }
	
	# Deny for accessing .htaccess files for Nginx
	location ~ /\.ht {
            deny all;
        }
	
	# Deny for accessing codes
        location ~ ^/(application|system|tests)/ {
            return 403;
        }
}
```

### Sub Directory Site Application


Codeiniter would guess your environment URI to implement pretty URL, if your Codeiniter application is under a sub folder of webroot, you just need to set the web server try file to current directory.

The following example would use `/_projects/codeigniter` as subdirectory path.

#### Application BaseUrl

From the orignal Codeigniter config file `application/config/config.php`:

```php
$config['base_url'] = '';
$config['index_page'] = 'index.php';
```

We could set subdirectory path to `base_url` and disable `index_page`:

```php
$config['base_url'] = '/_projects/codeigniter';
$config['index_page'] = '';
```

After setting, you would able to consistently bind URL when you use Url generator:

```php
$this->load->helper('url');
echo site_url('controller/action'); 	// `/_projects/codeigniter/controller/action`
echo base_url("controller/action");	// `/_projects/codeigniter/controller/action`	
redirect('controller/action');		// Go to `/_projects/codeigniter/controller/action`
```


#### Nginx Configuration

Customize the location path to current Codeiniter application directory:

```nginx
location /_projects/codeigniter/ {

	try_files $uri $uri/ /_projects/codeigniter/index.php;
}
```
