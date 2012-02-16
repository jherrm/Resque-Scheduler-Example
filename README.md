Rails Resque Scheduler Example
==============================

This is an example rails app that shows how to use resque + resque scheduler to run delayed jobs and scheduled jobs.

## Getting Started

**Dependencies**: Resque requires redis to run. Our initializer assumes redis is running locally. If you have another redis server running elsewhere, change it in `/config/initializers/resque.rb`.

Start the rails server and resque admin: `$ rails s`

- View the site at <http://localhost:3000>
- View the resque admin at <http://localhost:3000/resque>

Start the scheduler: `$ rake resque:scheduler`

- Every 30 seconds you should see a new job get added to the queue
    - `2012-02-17 16:19:59 queueing MyJob (do_my_job)`

Start the worker: `$ rake resque:work`

- Every 30 seconds you should see the output of the perform method in the MyJob class
    - `Doing my job`



## What's Inside?

### Required Gems

    gem 'resque' # background jobs
    gem 'resque-scheduler' # job scheduling

### Home for your job classes: `/app/jobs/`

    # Example: /app/jobs/myjob.rb

    module MyJob
      @queue = :my_job_queue
      def self.perform()
        # Do anything here, like access models, etc
        puts "Doing my job"
      end
    end

### Job Schedule

Define job schedules in `/config/resque_schedule.yml`:

    do_my_job:
      every: 30s
      class: MyJob
      args:
      description: Runs the perform method in MyJob

For ways to define delayed jobs and different schedules, [check here](https://github.com/bvandenbos/resque-scheduler).

### Initializer

`/config/initializers/resque.rb`: configures redis, link jobs folder and reads the schedule at `/config/resque_schedule.yml`.

### Rake tasks

Make the resque tasks available to rake and set default environment variables.

Rakefile: `require 'resque/tasks'`

Set the default queue environment variable: `/lib/tasks/resque.rake`:

    require 'resque/tasks'
    require 'resque_scheduler/tasks'

    task "resque:setup" => :environment do
      ENV['QUEUE'] = '*'
    end

### Resque admin

This code in `/config.ru` will run the resque admin site when you start the normal rails server:

    run Rack::URLMap.new \
      "/"       => ResqueScheduler::Application,
      "/resque" => Resque::Server.new
