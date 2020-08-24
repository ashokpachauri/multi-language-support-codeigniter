# multi-language-support-codeigniter

Multi-language support, also known as internationalization, is a key feature of modern web applications. Most of the full-stack PHP frameworks come with multi-language support which enables us to dynamically present our application’s interface in different languages without duplicating the existing source code for each language. Today we’re going to discuss how we can enable multiple languages using CodeIgniter as well as a few tricks to customize the core functionality. Configuring Multi-Language Support  First we need to configure the necessary files before we can start using language support. The CodeIgniter config file, located in the application/config directory, contains an option called language which defines the default language of the application.  

``` $config['language'] = 'english'; ```  

We also need to create the actual files which contain our messages in different languages. 

These files need to be placed inside the application/language directory with a separate directory for each language. 

For example, English language files should reside in the application/language/english directory, and French language files should be located in application/language/french.  

Let’s create some language files that contain error messages for a sample application. Create the file english/message_lang.php (it’s important that all of the language files have the suffix _lang.php). The following code contains some sample entries for the content of our language file:  
``` $lang["msg_first_name"] = "First Name";``` 

```$lang["msg_last_name"] = "Last Name";```

``` $lang["msg_dob"] = "Date of Birth";```

``` $lang["msg_address"] = "Address"; ```

 Of course, you can have multiple language files inside a single language directory. It’s recommended to group your messages into different files based on their context and purpose, and prefix your message keys with a file-specific keyword for consistency.  

Another way is to create separate message files for each controller. The advantage of this technique is that only the required messages are loaded instead of an entire language file which can create a certain level of performance overhead. 

Loading the Language Files  Even though we create language files, they are not effective until we load them inside controllers. The following code shows how we can load the files inside a controller:  

```class TestLanguage extends CI_Controller {```

      public function __construct() { 
          parent::__construct();  
              $this->lang->load("message","english");   
      }  
     function index() {      
          $data["language_msg"] = $this->lang->line("msg_hello_english"); 
          $this->load->view('language_view', $data);   
     }

```}```

We usually work with the language files within the controllers and views (using language files inside models is not such a good thing). Here we’ve used a controller’s constructor to load the language file so it can be used throughout the whole class, then we reference it in the class’ ```index()``` method.  

The first parameter to the ```lang->load()``` method will be the language’s filename without the _lang suffix. The second paramter, which is optional, is the language directory. It will point to default language in your config if it’s not provided here.  We can directly reference the entries of a language file using the ```lang->line()``` method and assign it’s return to the data passed into the view templates. 

Inside the view, we can then use the above language message as ```$language_msg```.  There may be an occasion when we need to load language files directly from the views as well. For example, using language items for form labels might be considered a good reason for directly loading and accessing messages inside views. 

It’s possible to use the same access method for these files inside views as inside controllers.  ``` $this->lang->line("msg_hello_english");```  Though it works perfectly, it could be confusing to have $this when our view template code isn’t an actual class. We can also use the following code with the support of the language helper to load language entries inside views, which gives us cleaner code.  

``` lang("msg_view_english");```  That’s basically all you need to know to get started working with CodeIgniter language files. But even though this is simple enough, it’s unnecessary and duplicated effort to load the necessary language files in each of the controllers, especially if your project contains hundreds of classes. 

Luckily, we can use CodeIgniter hooks to build a quick and effective solution for loading language files automatically for each controller. Assigning Language Loading Responsibilities to Hooks  CodeIgniter calls a few built-in hooks as part of its execution process. You can find a complete list of hooks in the user guide. 

We’ll use the post_controller_constructor hook which is called immediately after our controller is instantiated and prior to any other method calls.  We enable hooks in our application by setting the enable_hooks parameter in the main config file.  ``` $config['enable_hooks'] = TRUE;```  Then we can open the ```hooks.php``` file inside the config directory and create a custom hook as shown in the following code:  ``` $hook['post_controller_constructor'] = array(     'class' => 'LanguageLoader',     'function' => 'initialize',     'filename' => 'LanguageLoader.php',     'filepath' => 'hooks' );```

This defines the hook and provides the necessary information to execute it. The actual implementation will be created in a custom class inside the application/hooks directory.  ``` class LanguageLoader {     function initialize() {         $ci =& get_instance();         $ci->load->helper('language');         $ci->lang->load('message','english');     } }```

In here we don’t have the access to the language library using ```$this->lang```, so we need to get the CI object instance using the ```get_instance()``` function, and then we load the language as we did earlier. Now the language file will be available for every controller of our application without the need to manually load it inside the controllers. Learn PHP for free!  Make the leap into server-side programming with a comprehensive cover of PHP & MySQL.  

Normally RRP $11.95 Yours absolutely free Switching Between Different Languages  Once we have established support for multiple languages, a link for each language can be provided to the user, generally in one of our application’s menus, which the users can click and switch the language. A session or cookie value can be used to keep track of the active language.  

Let’s see how we can manage language switching using the hooks class we generated earlier. First we need to create a class to switch the language; we’ll be using a separate controller for this as shown below:  ``` class LangSwitch extends CI_Controller {     public function __construct() {         parent::__construct();         $this->load->helper('url');     }      function switchLanguage($language = "") {         $language = ($language != "") ? $language : "english";         $this->session->set_userdata('site_lang', $language);         redirect(base_url());     } }```

Then we need to define links to switch each of the available languages.  ```<a href='<?php echo $base_url; ?>langswitch/switchLanguage/english'>English</a> <a href='<?php echo $base_url; ?>langswitch/switchLanguage/french'>French</a>```

Whenever the user chooses a specific language, the ```switchLanguage()``` method of the LangSwitch class will assign the selected languages to the session and redirect the user to the home page.  Now the active language will be changed in the session, but still it won’t get affected until we load the specific language file for the active language. We also need to modify our hooks class to load the language dynamically from the session.  ``` class LanguageLoader {     function initialize() {         $ci =& get_instance();         $ci->load->helper('language');          $site_lang = $ci->session->userdata('site_lang');         if ($site_lang) {             $ci->lang->load('message',$ci->session->userdata('site_lang'));         } else {             $ci->lang->load('message','english');         }     } } ```

Inside the ```LanguageLoader``` class we get the active language and load the necessary language files, or we load the default language if the session key is absent. We can load multiple language files of a single language inside this class. Conclusion  Most of the full-stack PHP frameworks come with multi-language support which enables us to easily present our application’s interface in different languages. In this article we saw how to provide multiple languages using CodeIgniter. 

Of course, there are other ways of building up multi-language solutions, so feel free to discuss the best practices and experiences you had in implementing multi-language support in CodeIgniter and even in other frameworks. I’m looking forward to hearing from you!
