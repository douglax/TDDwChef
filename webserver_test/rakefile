# Rakefile

# -f correctness only fails the build on syntax violations.
# The warnings scan will still count and graph other violations.
desc 'Foodcritic correctness linting task'
task :foodcritic do
  sh 'foodcritic -f correctness .'
end

# --fail-level E will only fail Ruby errors.
# The warnings scan will still count and graph [W]arning and [C]op violations.
desc 'Rubocop linting task'
task :rubocop do
  sh 'rubocop --fail-level E'
end

# touchstone task
task jenkins: %w(rubocop foodcritic)
