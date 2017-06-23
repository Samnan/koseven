# Simple MVC Example

Let's get to see the simple way Kohana handle MVC, by creating out first controller, model and view and make them work together. In this example, we will create a simple page that fetches user reviews from database and displays them on the page.

## Model

First, we create a simple database model class that can keep our review data.

Create the file `application/classes/Model/Review.php` in your application folder and fill it out like so:

    <?php

	class Model_Review extends ORM {
	{
		/* this is optional */
		protected $_table_name = 'user_reviews';

		/* this is optional too */
		protected $_table_columns = array(
			'id'               =>  array('type'=>'int'),
			'posted_on'        =>  array('type'=>'datetime'),
			'rating'           =>  array('type'=>'int'),
			'username'         =>  array('type'=>'string'),
			'title'            =>  array('type'=>'string'),
			'comments'         =>  array('type'=>'string')
		);
	}

Lets see what's going on here:

`class Model_Review extends ORM {`
:	This line declares our model to extend from the kohana ORM module, which is the simplest ORM module provided with kohana, and works nicely for any starting project. Model class name has to be prefixed with `Model_` and an underscore delimited path to the folder the model is in (see [Conventions and styles](about.conventions) for more info).  Each model should also extend the base `ORM` or `Model` class which provides a standard way to query database. ORM module provides fastest way to do CRUD, so we use it in this example.

If the model class does not use `ORM`, but instead derives from `Model`, then we would simply do `class Model_Review extends Model {`.


`protected $_table_name = 'user_reviews';`
:	This line defines the name of the database table from which we are going to fetch user reviews. Note that we define the name of table here only because it is not named `reviews`. Kohana automatically will look for the table `reviews` in your database since the name of the model is `Model_Review` (note the pluralisation by Kohana). However, since our table name is different, so we elaborate how to use a different table name for a model, we explicitly define the table name to be `user_reviews` (note that no pluralisation does NOT take place if you define the table name yourself).

`protected $_table_columns = array(`
:	This definition is completely optional. What it does is explicitly define the table columns and their types for Kohana. If we remove this array from the model definition, then Kohana will automatically fetch the table column information from database when the model is used. Here we define the table column names just to save one extra query and also help understand the ORM a little better.

[!!] For most of the ORM model defintions, you would want to skip table column definitions inside the model. The reason for that is if your database tables are modified, then Kohana will automatically fetch all columns and work without any problems on its own. However, if you do define the table columns explicitly, then you have to make sure you update this array whenever you modify your table definition!

## Controller

Create the file `application/classes/Controller/Reviews.php` in your application folder and fill it out like so:

    <?php

	Class Controller_Reviews extends Controller_Template
	{
		public function action_index()
		{
			$this->template->content = View::factory('reviews');
			$model = ORM::factory('Review');
			$reviews = $model
						->where('rating', '>', 3)
						->order_by('posted_on', 'desc')
						->limit(10)
						->find_all();
			$this->template->content->reviews = $reviews;
		}
	}

Lets break down our controller:

`$this->template->content = View::factory('reviews');`
:	This line creates an instance of the view with the file 'reviews.php' inside the `views` folder. Notice that we have not
yet passed any arguments to the view, as we do that in the following lines.

`$model = ORM::factory('Review');`
: We create an instance of the `Model_Review` using `ORM::factory()` method. If the model class was derived from `Model`, then we would write the above as `$model = Model::factory('Review');`

		$reviews = $model
					->where('rating', '>', 3)
					->order_by('posted_on', 'desc')
					->limit(10)
					->find_all();

The above code is the meat of the page, which actually calls standard ORM methods to fetch reviews from database. Kohana ORM used chaining convention to simplify its usage, as you see in this example.

On the model instance, we first ask to filter the records where the user has rated the review with at least three stars or more. Then we sort the table results by the posted date (note the second optional parameter to sort in descending order). After that we limit to a maximum of 10 records and finally the `find_all()` call returns the actual results based on all the applied filters on the ORM class.


[!!] Since we are fetching multiple records here, we apply a limit ourselves, if we removed the `limit(10)` clause, then it will fetch all reviews from the database, which may dampen your memory usage. If you use the `find()` method of `ORM`, it automatically limits the results to one record.

`$this->template->content->reviews = $reviews;`
:	Eventually, we pass the reviews data to our view, by setting it as a variable of the `content` object. Remember that we set `content` to an instance of the view, so now we can pass it as many arguments before we render it. Also note that we do not explicitly render the view in our controller, that's because `Controller_Template` takes care of that formality by itself.

## View
Lets create the view which is simple HTML5 with some php code to display the reviews.

		<div class="reviews">
			<section class="clear">
				<?php foreach($reviews as $review) : ?>
				<article>
				<a name="u<?php echo $review->id; ?>"></a>
				<h3><?php echo HTML::chars(ucfirst($review->username)); ?>
					<strong><?php echo $review->title; ?></strong>
				</h3>
				<blockquote>
					<em><?php echo HTML::chars($review->comments); ?></em>
				</blockquote>
				</article>
				<?php endforeach; ?>
			</section>
		</div>

### Notes
As you see in the above view code, we iterate over each review, and display the data associated with it in each loop. Note the use `HTML::chars` to make sure any user data is escaped properly before showing in your browser to avoid security issues.

You may ask, where do we show the star rating for the review? Well, as a developer, you should now use this example and work on it to display the star rating on the view by yourself. Good Luck!
