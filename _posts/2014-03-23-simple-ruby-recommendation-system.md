---
layout: post
title:  "Simple recommendation system written in Ruby"
date:   2014-03-23 12:00:00
comments: true
categories: ruby
image: /images/008-recommend-ruby.jpg
---

I'm looking for new job; so yesterday I went through my old Rails projects, trying to describe them for my updated CV. I found interesting old project where I wrote recommendation system. Nothing fancy, just simple tag based recommendation for blog articles. I decided to extract some of the code and blog about it.

Algorithm used for recommendation is based on [Jaccard Index](http://en.wikipedia.org/wiki/Jaccard_index) and is also known as the Jaccard similarity coefficient. Jaccard Index was invented by botany professor [Paul Jaccard](http://en.wikipedia.org/wiki/Paul_Jaccard) and is basically a number that represents similarity and diversity of sample sets.

## How it works?

Take current item (blog post for example), and its properties that describe it the best (tags, categories or words). Then you take every other item and you calculate ```division``` of ```intersection``` and ```union```. Mathematical can this be represented with following equation.

![Jaccard Index - Wikipedia - Similarity](http://upload.wikimedia.org/math/1/8/6/186c7f4e83da32e889d606140fae25a0.png)

Equation will give you number between 0 and 1. You can also easily calculate dissimilarity between items - represented with next equation.

![Jaccard Index - Wikipedia - Dissimilarity](http://upload.wikimedia.org/math/0/2/9/02906c47e0a08707ad6e35a6c34a43b4.png)

## Example implementation with Ruby

I'll be recommending books based on words in titles. So simple ```Book``` class will do the job just fine.

```ruby
class Book < Struct.new(:title)

  # Array of unique words longer than 2 characters.
  # Could also be array of tags or categories.
  def words
    @words ||= self.title.gsub(/[a-zA-Z]{3,}/).map(&:downcase).uniq.sort
  end

end
```

```BookRecommender``` class is initialized using current book and array of books. The method ```recommendations``` will loop over array and set ```jaccard_index``` for each item; then items will be sorted accordingly.

```ruby
class BookRecommender

  def initialize book, books
    @book, @books = book, books
  end

  def recommendations

    # Map jaccard_index to each item and sort array
    @books.map! do |this_book|

      # We can define jaccard_index getter in runtime as singleton...
      this_book.define_singleton_method(:jaccard_index) do
        @jaccard_index
      end

      # also setter
      this_book.define_singleton_method("jaccard_index=") do |index|
        @jaccard_index = index || 0.0
      end

      # Calculate intersection between sets
      intersection = (@book.words & this_book.words).size
      # ... and union
      union = (@book.words | this_book.words).size

      # Assign the division and rescue if division is not possible with 0
      this_book.jaccard_index = (intersection.to_f / union.to_f) rescue 0.0

      this_book

      # Sort items
    end.sort_by { |book| 1 - book.jaccard_index }

  end

end
```

For demonstration purposes...

```ruby
# ...
# Read data and define array of books
BOOKS = DATA.read.split("\n").map { |l| Book.new(l) }

# Define current book
current_book = Book.new("Ruby programming language")

# Do recommendation...
books = BookRecommender.new(current_book, BOOKS).recommendations

books.each do |book|
  puts "#{book.title} (#{'%.2f' % book.jaccard_index})"
end

__END__
Finding the best language for the job
Could Ruby save the day
Python will rock your world
Is Ruby better than Python
Programming in Ruby is fun
Python to the moon
Programming languages of the future
```

Sample output for title "Ruby programming language" with Jaccard index visible on the right side.

```
Programming in Ruby is fun (0.50)
Programming languages of the future (0.17)
Is Ruby better than Python (0.17)
Could Ruby save the day (0.14)
Finding the best language for the job (0.12)
Python to the moon (0.00)
Python will rock your world (0.00)
```


## Conclusion
This is purely Ruby based solution - [source code can be found on my gist](https://gist.github.com/otobrglez/9738998). I've also written [PostgreSQL versions](https://gist.github.com/otobrglez/1078953) that uses tags. Please have in mind that if the sets are bigger the code will get slower; so don't do recommendations each time you "open" item. Define background service that will calculate recommendations in background!

Hope that you can use this in your projects and please - give me some feedback. Thanks! :)

