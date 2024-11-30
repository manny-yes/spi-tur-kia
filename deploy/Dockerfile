FROM ruby:3.2.6

RUN apt-get update -qq && apt-get install -y \
    build-essential \
    libpq-dev \
    postgresql-client \
    nodejs \
    curl \
    tar

# Set working directory
WORKDIR /app

# Copy the source from the release
COPY src/. ./.

# Fix for pg error -- error: use of undeclared identifier 'rb_cData'
RUN bundle update pg

# Install gems
RUN bundle install

# Precompile assets for production
RUN bundle exec rake app:assets:precompile

# Expose the Rails default port
EXPOSE 3000

# Default bash
CMD ["bash"]

