HTML Purifier & Markdown Behavior for Yii
===
Contains methods for Markdown parser and HTML Purifier for Yii ActiveRecord models.

HTML Purifier and Markdown parser very powerfull, but very slow. We can use two fields in a database (for sourse text and for processed text) and process a content before saving of a model.

This behavior parses a content from a sourse field to a destination field automatically (if a destination field has empty value). By default a behavior processes content in the `beforeSave()` and `afterFind()` events. 

Readme
---

[README RUS](http://www.elisdn.ru/blog/12/dpurifytextbehavior-ispolzuem-html-purifier-dlia-filtracii-dannih-v-yii)

Installation
------------

Extract to `protected/components`. Create two fileds like `text` and `purified_text`(or `source_text` and `text` for exmple) in your model. Attach the Behavior to your model.

Usage example
-------------
 
Let's allow insert Flash-video blocks and attribute `rel="nofollow"` for links in our Blog posts (also we can activate Markdown Parser if needle):

~~~
[php]
class Post extends CActiveRecord
{
    public function behaviors()
    {
        return array(
            'PurifyText'=>array(
                'class'=>'DPurifyTextBehavior',
                'sourceAttribute'=>'text',
                'destinationAttribute'=>'purified_text',
                // 'enableMarkdown'=>true,
                'purifierOptions'=> array(
                    'Attr.AllowedRel'=>array('nofollow'),
                    'HTML.SafeObject'=>true,
                    'Output.FlashCompat'=>true,
                ),
                // 'updateOnAfterFind'=>false  
            ),
        );
    }
}
~~~

Let's set more strong rules for Comments:

~~~
[php]
class Comment extends CActiveRecord
{
    public function behaviors()
    {
        return array(
            'PurifyText'=>array(
                'class'=>'DPurifyTextBehavior',
                'sourceAttribute'=>'text',
                'destinationAttribute'=>'purified_text',
                'purifierOptions'=>array(
                    'AutoFormat.AutoParagraph' => true,
                    'HTML.Allowed'=>'p,ul,li,b,i,a[href],pre',
                    'AutoFormat.Linkify'=>true,
                    'HTML.Nofollow'=>true,
                    'Core.EscapeInvalidTags'=>true,
                ),              
            )
        );
    }
}
~~~

Now we must use the `purified_text` attribute in views:

~~~
[php]
<?php echo $model->purified_text; ?>
~~~

And it returns a processed HTML text from our database.

The behavior also automatically converts content and saves result to database by a `onAfterFind` event if result field is empty. For experiments with a purifier options you can clean results in a database and you can add line `'updateOnAfterFind'=>false`. 

Also if you want to regenerate all your processed texts you can execute query

~~~
[sql]
UPDATE `posts` SET `purified_text` = '';
~~~

and all your entries will be updated automatically by the first `onAfterFind` event. 

P.S. If you use Markdown, place string

~~~
[php]
CTextHighlighter::registerCssFile();
~~~

in your view for syntax hightlighting work.