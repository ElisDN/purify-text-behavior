HTML Purifier & Markdown Behavior for Yii
===
Contains methods for Markdown parser and HTML Purifier for Yii ActiveRecord models.

HTML Purifier and Markdown parser very powerfull, but very slow. We can use two fields in database (for sourse text and for processed text) and process content before saving of model.

This behavior parses content from sourse field to destination field automatically if destination field has empty value. By default a behavior purify content in `beforeSave()` and `afterFind()` events. 

Readme
---

[README RUS](http://www.elisdn.ru/blog/12/dpurifytextbehavior-ispolzuem-html-purifier-dlia-filtracii-dannih-v-yii)

Installation
------------

Extract to `protected/components`. Create two fileds like `text` and `purified_text`(or `source_text` and `text` for exmple) in your model. Attach the Behavior to your model.

Usage example
-------------
 
Let's allow Flash-video insertions and allow attribute `rel="nofollow"` for links in our Blog posts (also we can activate Markdown Parser if needle):

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

Now we must use `purified_text` attribute in views:

~~~
[php]
<?php echo $model->purified_text; ?>
~~~

And it returns processed HTML text from database.

By default a behavior also automatically converts content and saves result to database by `onAfterFind` event if result field is empty. For experimentation with purifier options you can clean results in database and can add line `'updateOnAfterFind'=>false`. 

Also if you want regenerate all your processed content you can execute query

~~~
[sql]
UPDATE `posts` SET `purified_text` = '';
~~~

and all entries automatically updates by first `onAfterFind` event. 

P.S. If you use Markdown, place string

~~~
[php]
CTextHighlighter::registerCssFile();
~~~

in your view for enable syntax hightlighting.