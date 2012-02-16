Rails Resque Scheduler Example
==============================

This is an example rails app that shows how to use resque + resque scheduler to run delayed jobs and scheduled jobs.

## What's Inside?


### New directory for your job classes: `/app/jobs/`

Holds classes like `myjob.rb`:

    module MyJob
      @queue = :my_job_queue
      def self.perform()
        # Do anything here, like access models, etc
        puts "Doing my job"
      end
    end

### Required Gems

    gem 'resque' # background jobs
    gem 'resque-scheduler' # job scheduling


### Rake tasks

Rakefile:

    require 'resque/tasks'

Sets the default queue environment variable: `/lib/tasks/resque.rake`:

    require 'resque/tasks'
    require 'resque_scheduler/tasks'

    task "resque:setup" => :environment do
      ENV['QUEUE'] = '*'
    end


### Job Schedule

Define job schedules: `/config/resque_schedule.yml`:

    do_my_job:
      every: 30s
      class: MyJob
      args:
      description: Runs the perform method in MyJob


### Initializers

Configures redis, link jobs folder and reads schedule: `/config/initializers/resque.rb`



### Resque admin

When starting the rails server, run the resque admin site with it: `/config.ru`:

    run Rack::URLMap.new \
      "/"       => ResqueScheduler::Application,
      "/resque" => Resque::Server.new



## Getting Started

Start the rails server and resque admin: `$ rails s`

Start the scheduler: `$ rake resque:scheduler`

Start the worker: `$ rake resque:work`
