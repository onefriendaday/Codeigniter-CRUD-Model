#CodeIgniter-CRUD-Model

Be aware this is not meant to be an ORM library. It is a simple base model which 
your models can extend to give them supercharged features. This model doesn't
require you to change how you would natively interact with CodeIgniter's database
or Active Record class very much. It's not designed to help writing your database
interactions more efficient, not to interfere.

##Installation

Save MY_Model.php as application/core/MY_Model.php

##Creating a Model

To demonstrate how to use this base model, we'll work with an imaginary database
table called "posts" which has 5 fields: post_id, user_id, post_date,
post_title and post_content. Let's look at the basic setup for this model:

At the very minimum, a model extending MY_Model must declare the table name
and primary key:

    <?php

    class Posts extends MY_Model {

        // You must always define the table and the primary key
        public $table = 'posts';
        public $primary_key = 'posts.post_id';

    }

This is all it takes to perform basic CRUD actions against this model.

##Basic CRUD

### Selecting Records

    <?php

    // Load the model
    $this->load->model('mdl_posts');

    // Get all the posts
    $posts = $this->mdl_posts->get()->result();

    // Get a post by ID
    $posts = $this->mdl_posts->get_by_id(3);

    // Get posts meeting criteria:
    $posts = $this->mdl_posts->where('user_id', 5)->get()->result();

    // Set an order by
    $posts = $this->mdl_posts->order_by('post_date DESC')->get()->result();

As you can tell, there isn't much difference in the syntax between our model
calls and calls to CodeIgniter's native DB class. The idea isn't to reinvent 
anything - instead it's to make things simpler and more efficient. Any of
CodeIgniter's native active record functions can be chained to our model calls,
and are used exactly how they would be as if they were called from $this->db.

###Inserting Records

If your form fields are named exactly as the database fields are, inserting 
new records into the database is as simple as using the save function inside a
controller after the form has been submitted:

    $this->mdl_posts->save();

The base model automatically creates the array of data for you based on what is
posted from the form.

###Updating Records

Updating existing records is exactly the same as inserting records, except
the save function needs the id of the record to update:

    $this->mdl_posts->save(12);

###Deleting Records

The base model's delete function accepts the record's id for deletion:

    $this->mdl_posts->delete(5);

##Form Validation

Let's add validation to our model by creating a function called validation_rules
and using it to return an array of CodeIgniter validation rules:

    <?php

    class Mdl_Posts extends MY_Model {

        // You must define both the table and primary key
        public $table = 'posts';
        public $primary_key = 'posts.post_id';

        public function validation_rules()
        {
            return array(
                'user_id' => array(
                    'field' => 'user_id',
                    'label' => 'Author',
                    'rules' => 'required'
                ),
                'post_date' => array(
                    'field' => 'post_date',
                    'label' => 'Post Date',
                    'rules' => 'required'
                ),
                'post_title' => array(
                    'field' => 'post_title',
                    'label' => 'Post Title',
                    'rules' => 'required'
                ),
                'post_content' => array(
                    'field' => 'post_content',
                    'label' => 'Post Content',
                    'rules' => 'required'
                )
            );
        }

    }

Now, we can run validation on the model when submitting a form from our
controller:

    if ($this->mdl_posts->run_validation())
    {
        // The form validated
    }
    else
    {
        // The form didn't validate
    }

Your application may require that a single model be able to validate against 
different sets of rules based on the action. For example, maybe editing a post
should have different validation rules than creating a new post. In that case,
create another validation function in your model, call it whatever you want,
and specify the name of the function in the run_validation() call in your 
controller:

    if ($this->mdl_posts->run_validation('validation_rules_edit'))
    {
        // The form validated
    }
    else
    {
        // The form didn't validate
    }

This example would use the validation rule array returned from the function 
called validation\_rules\_edit in your post model.

##Form Pre-Population

Pre-populating the form based in an existing database record is simple:

In your controller:

    public function edit($id = NULL)
    {
        if (!$_POST)
        {
            $this->mdl_posts->prep_form($id);
        }

        $this->load->view('post_form');

Note that we only want the prep_form() function to be called under the condition
that the form has not yet been submitted. In the form view, set the values in 
your form fields using the model's form_value() function:

    <input type="text" name="post_title" value="<?php echo $this->mdl_posts->form_value('post_title'); ?>">
    
    <textarea name="post_content"><?php echo $this->mdl_posts->form_value('post_content'); ?></textarea>

The form_value() function will pre-populate your form when editing an existing
record and will also retain your previously submitted values in case the form
doesn't validate upon submission.

##Additional Features

###Modifying Data Pre-Insert/Update

The base model contains a db_array() function which creates an array to submit
to your database based on posted values from a form. Oftentimes, we need to do 
things to that data before it actually gets to the database.

The easiest way to do this is to create a db_array() function inside the model,
grab the base model's array, modify it, and return it.

For our example, let's say that the post form displays the date in a human
readable form, but we need to save it as a UNIX timestamp:

    public function db_array()
    {
        // Grab the default array from the base model
        $db_array = parent::db_array();

        // Change the post date to a UNIX timestamp
        $db_array['post_date'] = strtotime($db_array['post_date']);

        // Return the array
        return $db_array;
    }

Now, any time your application calls $this->mdl_posts->save(), the data will be
modified before it reaches the database.

###Modifying Data Pre-Form Population

In the example given above, we're storing the date as a UNIX timestamp, but we
need to display the date as a human readable value in the form. To do this,
create a prep_form() function inside the model and modify the date value:

    public function prep_form($id = NULL)
    {
        if ($id)
        {
            // This is for an existing record

            // Have the base model do the initial form preparation
            parent::prep_form($id);

            // Change the post date from a unix timestamp to a formatted value
            $this->set_form_value('post_date', date('m/d/Y', $this->form_value('post_date')));
        }
        else
        {
            // This is for a new record

            // Set the date to current date by default
            $this->set_form_value('post_date', date('m/d/Y');
        }
    }

And in our view:

    <input type="text" name="post_date" value="<?php echo $this->mdl_posts->form_value('post_date'); ?>">

This example will output the date in the form formatted as m/d/Y, using the 
date value in the database if it's an existing record, or using today's date
as default if it's a new record.

###Filters

Let's say we want to grab all the posts belonging to a specific user. We can do
it pretty easily like this:

    $this->mdl_posts->where('user_id', 5)->get()->result();

That's easy enough, but maybe there are multiple points within your application
that need to retrieve posts by the user id. In this case, we can add a function
to our model as such:

    public function by_user($user_id)
    {
        $this->filter_where('user_id', $user_id);
        return $this;
    }

And in our controller:

    // Instead of calling this:
    $this->mdl_posts->where('user_id', 5)->get()->result();

    // We'll call this:
    $this->mdl_posts->by_user(5)->get()->result();

The filter function added to the model can be named anything. The filter_where()
function call is simply a CodeIgniter Active Record function prefixed with "filter_".
Any CodeIgniter Active Record function can be used as a filter.

###Model Defaults

There are many cases where when records are retrieved from a database, the results
are expected to *always* be returned in a certain way.

    <?php

    class Mdl_Posts extends MY_Model {

        // You must define both the table and primary key
        public $table       = 'posts';
        public $primary_key = 'posts.post_id';

        public function default_select()
        {
            $this->db->select('posts.*, users.username');
        }

        public function default_join()
        {
            $this->db->join('users', 'users.user_id = posts.user_id');
        }

        public function default_order_by()
        {
            $this->db->order_by('posts.post_date DESC');
        }

Just like the filters, the default functions are the same as any CodeIgniter 
Active Record function, except they're prefixed with "default_".

So, instead of doing this:

    $posts = $this->mdl_posts->select('posts.*, users.username')->join('users', 'users.user_id = posts.user_id')->order_by('posts.post_date DESC')->get()->result();

We can simply do this:

    $posts = $this->mdl_posts->get()->result();

###Pagination

Pagination can be performed very easily from your controller:

    public function index($page = 0)
    {
        // Initiate the pagination
        $this->mdl_posts->paginate(site_url('posts/index'), $page);
        
        // Collect the results
        $posts = $this->mdl_posts->result();
        
        // Create the array to pass to the view
        $data = array(
            'posts' => $posts
        );
        
        // Load the view
        $this->load->view('post_index', $data);
    }

This allows us to paginate results using only two lines of code. The paginate()
function requires two parameters - the base URL to construct the paginated links
and the page.

To display the page links in the view:

    <?php echo $this->mdl_posts->page_links; ?>

The style in which the links are generated can be specified by creating a config
file and loading the pagination_style config item in your application before 
the pagination occurs. The file may be named anything, but the config item
needs to be named "pagination_style". Here's an example pagination config file:

    <?php

    // Located in application/config/pagination_style.php (as an example)

    $config = array(

        'pagination_style'		=> array(
            'first_link'		=> '&lsaquo;&lsaquo;',
            'next_link'			=> '&rsaquo;',
            'prev_link'			=> '&lsaquo;',
            'last_link'			=> '&rsaquo;&rsaquo;',
            'full_tag_open'		=> '<div class="pagination"><ul>',
            'full_tag_close'	=> '</ul></div>',
            'first_tag_open'	=> '<li>',
            'first_tag_close'	=> '</li>',
            'last_tag_open'		=> '<li>',
            'last_tag_close'	=> '</li>',
            'cur_tag_open'		=> '<li class="active"><a href="#">',
            'cur_tag_close'		=> '</a></li>',
            'next_tag_open'		=> '<li>',
            'next_tag_close'	=> '</li>',
            'prev_tag_open'		=> '<li>',
            'prev_tag_close'	=> '</li>',
            'num_links'			=> '10'
        )

    );
    ?>

##Finish

I hope you find this guide and the base model useful. It's been crazy useful for
me, and I wouldn't dare consider starting any CodeIgniter project without it.
Happy coding!