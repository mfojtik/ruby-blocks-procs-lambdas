!SLIDE

    @@@ ruby
    ['blocks', 'Proc', 'lambdas'].map do |i|
      puts i
    end
    puts "in Ruby"

!SLIDE
# Basics #

!SLIDE small
# Syntax sugar #
    @@@ ruby

    def names
      yield "Peter", "Bob"
    end

    names { |x,y| puts x,y } # => [ 'Peter', 'Bob' ]

    names do |x,y|
      puts x, y
    end => [ 'Peter', 'Bob' ]

!SLIDE small
# A "real world" example #

    @@@ ruby
    def write_to_file(filename)
      f = File.open(filename, 'w')
      yield f
      f.close
    end

    write_to_file "test.txt" { |f| f.write "Hello!" }

!SLIDE
# Fun with Array #

    @@@ ruby
    array = [1, 2, 3, 4]

    array.collect do |n|
      n ** 2
    end # => [1, 4, 9, 16]

!SLIDE
    @@@ ruby
    array = [1, 2, 3, 4]

    array.collect! do |n|
      n ** 2
    end

    puts array.inspect # => [1, 4, 9, 16]

!SLIDE
# Implementing similar method for Array

!SLIDE smaller
    @@@ ruby
    class Array
      def iterate!
        self.each_with_index do |n, i|
          self[i] = yield(n)
        end
      end
    end

    array = [1, 2, 3, 4]
    array.iterate! do |n|
      n ** 2
    end

    puts array.inspect # => [1, 4, 9, 16]

!SLIDE smaller
# Proc.new #

    @@@ ruby
    class Array
      def iterate!(code)
        self.each_with_index do |n, i|
          self[i] = code.call(n)
        end
      end
    end

    code = Proc.new { |n| n ** 2 }
    array = [1, 2, 3, 4]

    array.iterate!(code)
    puts array.inspect # => [1, 4, 9, 16]

!SLIDE smaller
# Callback (:after, :before)

    @@@ ruby
    def process_image(image, options={})
      options[:before].call(image)
      puts "Do something with image..."
      options[:after].call(image)
    end

    process_image(image_data,
      :before => Proc.new { puts "Checking image format..." },
      :after => Proc.new { puts "Upload image to the server..." }
    )

    # Checking image format...
    # Do something with image...
    # Upload image to the server...

!SLIDE bullets incremental
# blocks or Procs? #

* block: Your method is breaking an object down into smaller pieces, and you want to let your users interact with these pieces.
* Proc: You want to reuse a block of code multiple times.
* Proc: Your method will have one or more callbacks.

!SLIDE
# Lambdas #

!SLIDE
    @@@ ruby
    class Array
      def iterate!(code)
        self.each_with_index do |n, i|
          self[i] = code.call(n)
        end
      end
    end

    array = [1, 2, 3, 4]

    array.iterate!(lambda { |n| n ** 2 })

    puts array.inspect

    # => [1, 4, 9, 16]

!SLIDE
# Difference 1. #

## Lambdas check the number of arguments passed

!SLIDE smaller
    @@@ ruby
    def args(code)
      one, two = 1, 2
      code.call(one, two)
    end

    args(Proc.new{|a, b, c| " #{a} and #{b} and #{c.class}"})
    # => Give me a 1 and a 2 and a NilClass

    args(lambda{|a, b, c| " #{a} and #{b} and #{c.class}"})
    # *.rb:8: ArgumentError: wrong number of arguments
    #   (2 for 3) (ArgumentError)

!SLIDE
# Difference 2. #

## Lambdas have diminutive returns

!SLIDE smaller
    @@@ ruby
    def proc_return
      Proc.new { return "Proc.new"}.call
      return "proc_return method finished"
    end

    def lambda_return
      lambda { return "lambda" }.call
      return "lambda_return method finished"
    end

    puts proc_return
    puts lambda_return

    # => Proc.new
    # => lambda_return method finished

!SLIDE smaller
# Lambdas or Procs? #

    @@@ ruby
    def generic_return(code)
      value = code.call
      "generic_return finished => #{value}"
    end

    puts generic_return(
      Proc.new { return "Proc.new" }
    )

    puts generic_return(
      lambda { return "lambda" }
    )

    # => *.rb:6: unexpected return (LocalJumpError)
    # => generic_return method finished => lambda


!SLIDE
# Conclusion #

* blocks and Proc act like drop-in code snippets
* lambdas act just like normal methods

!SLIDE smaller
# Lazy bitch excersise
Carla is a really lazy bitch. She had three customers last month. Since Carla is a skilled Ruby programmer, she put them into to the Array:
    @@@ruby
    customers = [
      ["Peter", 1000], ["Bob", 4000], ["Mike", 6000]
    ]
Carla wants to know how much she earned. Her pimp takes 30% provision for those customers who pay more than 3000$ and 10% for those who pay less.
Carla wants to improve her programming skills, so she wants to use Ruby blocks to do all calculations.

!SLIDE smaller
# (Possible :) solution

    @@@ ruby
    customers = [
      ["Peter", 1000], ["Bob", 4000], ["Mike", 6000]
    ]

    class Array
      def count_money
        self.each do |customer|
          if customer[1] > 3000
            yield customer[1] - (customer[1] * 0.30)
          end
          if customer[1] <= 3000
            yield customer[1] - (customer[1] * 0.10)
          end
        end
      end
    end

    total_cash = 0
    customers.count_money { |cash| total_cash+=cash }
    puts total_cash # => 7900.0

!SLIDE smaller
# Carla graduated
Now Carla become a real programmer and she also has more pimps in different
countries. Now she need to improve her algorithm to support multiple provision
calculation.

* Pimp David took **15%** provision
* Pimp Jose took **20%** provision

## Carla created a Hash:

    @@@ ruby
    customers = {
      :david =>  [
        ["Peter", 1000], ["Bob", 4000], ["Mike", 6000]
      ],
      :jose =>  [
        ["Mark", 3000], ["Stan", 2000], ["Kyle", 1000]
      ]
    }

!SLIDE smaller
    @@@ ruby
    customers = {
      :david =>  [
        ["Peter", 1000], ["Bob", 4000], ["Mike", 6000]
      ],
      :jose =>  [
        ["Mark", 3000], ["Stan", 2000], ["Kyle", 1000]
      ]
    }

    class Array
      def money(code)
        self.each do |customer|
          yield code.call(customer[1])
        end
      end
    end

    david = Proc.new { |money| money - (money * 0.15) }
    jose = Proc.new { |money| money - (money * 0.20) }

    total_cash = 0
    customers[:david].money(david) { |cash| total_cash+=cash }
    customers[:jose].money(jose) { |cash| total_cash+=cash }
    puts total_cash # => 14150.0
