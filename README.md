# evote-movie-2019-16-login-protect-crud

Part of the progressive Movie Voting website project at:
https://github.com/dr-matt-smith/evote-movie-2019

The project has been refactored as follows:

- create a new class `/src/SessionManager.php`:

    ```php
        namespace Tudublin;
        
        class SessionManager
        {
            private $username = '';
            private $loggedIn = false;
            
            public function __construct()
            {
                if(isset($_SESSION['username'])){
                    $this->username = $_SESSION['username'];
                    $this->loggedIn = true;
                }
            }
            
            public function isLoggedIn()
            {
                return $this->loggedIn;
            }
            
            public function usernameFromSession()
            {
                return $this->username;
            }
        }
    ```
- create a new class `/src/LoginController.php`:

    ```php
        namespace Tudublin;
        
        class LoginController
        {
            function loginForm($error = false)
            {
                $pageTitle = 'login';
                if($error){
                    $errorMessage = 'invalid credentials - please try again';
                }
                require_once __DIR__ . '/../templates/login.php';
            }
            
            public function processLogin()
            {
                $username = filter_input(INPUT_POST, 'username');
                $password = filter_input(INPUT_POST, 'password');
        
                if($this->validCredentials($username, $password)){
                    $this->successfulLoginAction($username);
                } else {
                    // redisplay login form - but with an error message
                    $this->loginForm(true);
                }
            }
        
            public function validCredentials($username, $password)
            {
                if('admin' == $username && 'admin' == $password){
                    return true;
                }
        
                if('matt' == $username && 'smith' == $password){
                    return true;
                }
        
                return false;
            }
        
            private function successfulLoginAction($username)
            {
                // session stuff here
        
                $_SESSION['username'] = $username;
                $mainController = new MainController();
                $mainController->home();
            }
        
            public function logout()
            {
                $_SESSION = [];
                $mainController = new MainController();
                $mainController->home();
            }
        }

    ```

- create new template of Login form: `templates/login.php`

    ```php
        <?php
        require_once __DIR__ . '/_header.php';
        ?>
        
        <?php
        if(!empty($errorMessage)):
        ?>
            <div style="background-color: pink">
                <?= $errorMessage ?>
            </div>
        <?php
        endif;
        ?>
        
        <form action="index.php" method="POST">
            <input type="hidden" name="action" value="processLogin">
        
            <p>
            Username:
            <input name="username">
            <p>
                Password:
                <input type="password" name="password">
        
            <p>
            <input type="submit" value="LOGIN">
        
        </form>
        
        <?php
        require_once __DIR__ . '/_footer.php';
        ?>
    ```
    
- update template 'templates/_header.php' to display login/logout links depending on whether user is logged in (variables for `$isLoggedIn` and `$username` are set at the beginning of this script from a `SessionManager` object):

    ```php
      <?php
      //----- login variables
      use Tudublin\SessionManager;
      $sessionManager = new SessionManager();
      $isLoggedIn = $sessionManager->isLoggedIn();
      $username = $sessionManager->usernameFromSession();
      
      // ------ page title
      if(empty($pageTitle)){
          $pageTitle = '';
      }
      
      // ----- current page nav styles
      if(empty($homePageStyle)) $homePageStyle = '';
      if(empty($aboutPageStyle)) $aboutPageStyle = '';
      if(empty($contactPageStyle)) $contactPageStyle = '';
      if(empty($sitemapPageStyle)) $sitemapPageStyle = '';
      if(empty($listPageStyle)) $listPageStyle = '';
      if(empty($listCheapPageStyle)) $listCheapPageStyle = '';
      ?>
      <!doctype html>
      <html lang="en">
      <head>
          <title>EVOTE MOVIE - <?= $pageTitle ?> page</title>
          <meta charset="utf-8">
          <style>
              @import "/css/basic.css";
              @import "/css/nav.css";
              @import "/css/footer.css";
          </style>
      </head>
      <body>
      
      
      
      <header>
          <!-- LOGIN LINK -->
          <span style="float: right; text-align: right;">
              <?php
                  if($isLoggedIn):
              ?>
                  You are logged in as: <strong><?= $username ?></strong>
                  <br>
                  <a href="index.php?action=logout">Logout</a>
              <?php
                  else:
              ?>
                  <a href="index.php?action=login">Login</a>
              <?php
                  endif;
              ?>
          </span>
      
          <!-- LOGO -->
          <img src="/images/smithit_logo.gif" alt="logo">
      </header>
    
      ... as before

    ```

- create new template `templates/_list.php`, to contain the body of the movie list for the list and cheap movies pages. Note the use of `IF` statements to hide edit/delete/new options unless user is logged in

    ```php
        <table>
            <tr>
                <th> ID </th>
                <th> title </th>
                <th> price </th>
        
                <?php
                if($isLoggedIn):
                ?>
                    <th> &nbsp; </th>
                    <th> &nbsp; </th>
                <?php
                    endif;
                ?>
            </tr>
        
            <?php
                foreach($movies as $movie):
            ?>
        
                    <tr>
                        <td><?= $movie->getId() ?></td>
                        <td><?= $movie->getTitle() ?></td>
                        <td>&euro; <?= $movie->getPrice() ?></td>
        
                        <?php
                            if($isLoggedIn):
                        ?>
                        <td>
                            <a href="index.php?action=deleteMovie&id=<?= $movie->getId() ?>">DELETE</a>
                        </td>
                        <td>
                            <a href="index.php?action=editMovie&id=<?= $movie->getId() ?>">EDIT</a>
                        </td>
                        <?php
                            endif;
                        ?>
                    </tr>
        
            <?php
                endforeach;
            ?>
        
        </table>
        
        <?php
        if($isLoggedIn):
        ?>
            <hr>
            <a href="index.php?action=newMovieForm">new movie form</a>
        <?php
        endif;
        ?>
        
        
        <?php
        require_once __DIR__ . '/_footer.php';
        ?>
    ```
    
- edit templates `templates/list.php` to make use of the new partial `_list.php`:


    ```php
      <?php
      require_once __DIR__ . '/_header.php';
      ?>
      
      <!-- start table for displaying DVD details -->
      <h2>Lists of movies and their average votes</h2>
      
      <?php
      require_once __DIR__ . '/_list.php';
      ?>
    ```
    
- edit templates `templates/listCheap.php` to make use of the new partial `_list.php`:

    ```php
      <?php
      require_once __DIR__ . '/_header.php';
      ?>
      
      <!-- start table for displaying DVD details -->
      <h2>Cheap movies !!!</h2>
      
      <?php
      require_once __DIR__ . '/_list.php';
      ?>
    ```
    
    
- edit the Front Controller to start a session, create an instance-object of class `LoginController`, and add some new route actions for login/logout/processLogin:

    ```php
      <?php
      session_start();
      
      error_reporting(E_ALL);
      ini_set('display_errors', 1);
      
      require_once __DIR__ . '/../config/db.php';
      require_once __DIR__ . '/../vendor/autoload.php';
      
      use Tudublin\MainController;
      use Tudublin\AdminController;
      use Tudublin\LoginController;
      
      // get action GET parameter (if it exists)
      $action = filter_input(INPUT_GET, 'action', FILTER_SANITIZE_STRING);
      if(empty($action)){
          $action = filter_input(INPUT_POST, 'action', FILTER_SANITIZE_STRING);
      }
      
      // based on value (if any) of 'action' decide which template to output
      $mainController = new MainController();
      $adminController = new AdminController();
      $loginController = new LoginController();
      switch ($action){
          // ------ login section --------
          case 'login':
              $loginController->loginForm();
              break;
      
          case 'logout':
              $loginController->logout();
              break;
      
          case 'processLogin':
              $loginController->processLogin();
              break;
      
      
          // ----- public pages ----
          case 'about':
              $mainController->about();  

        ... as before
    ```
    
- some other minor changes were made, relating to a new folder `admin` created in `templates`:

    - the 2 admin forms were put into `templates/admin/`
    
    - the paths in `AdminController` were updatesd to reflect the new form locations
    
    - the 2 admin form templates had their `_header/_footer` relative paths updated