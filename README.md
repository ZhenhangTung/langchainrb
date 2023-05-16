🦜️🔗 LangChain.rb
---
⚡ Building applications with LLMs through composability ⚡

👨‍💻👩‍💻 CURRENTLY SEEKING PEOPLE TO FORM THE CORE GROUP OF MAINTAINERS WITH

:warning: UNDER ACTIVE AND RAPID DEVELOPMENT (MAY BE BUGGY AND UNTESTED)

![Tests status](https://github.com/andreibondarev/langchainrb/actions/workflows/ci.yml/badge.svg) [![Gem Version](https://badge.fury.io/rb/langchainrb.svg)](https://badge.fury.io/rb/langchainrb)

Langchain.rb is a library that's an abstraction layer on top many emergent AI, ML and other DS tools. The goal is to abstract complexity and difficult concepts to make building AI/ML-supercharged applications approachable for traditional software engineers.

## Installation

Install the gem and add to the application's Gemfile by executing:

    $ bundle add langchainrb

If bundler is not being used to manage dependencies, install the gem by executing:

    $ gem install langchainrb

## Usage

```ruby
require "langchain"
```

#### Supported vector search databases and features:

| Database | Querying           | Storage | Schema Management | Backups | Rails Integration | ??? |
| -------- |:------------------:| -------:| -----------------:| -------:| -----------------:| ---:|
| Weaviate | :white_check_mark: | WIP     | WIP               | WIP     |                   |     |
| Qdrant   | :white_check_mark: | WIP     | WIP               | WIP     |                   |     |
| Milvus   | :white_check_mark: | WIP     | WIP               | WIP     |                   |     |
| Pinecone | :white_check_mark: | WIP     | WIP               | WIP     |                   |     |

### Using Vector Search Databases 🔍

Choose the LLM provider you'll be using (OpenAI or Cohere) and retrieve the API key.

Add `gem "weaviate-ruby", "~> 0.8.0"`  to your Gemfile.

Pick the vector search database you'll be using and instantiate the client:
```ruby
client = Vectorsearch::Weaviate.new(
    url: ENV["WEAVIATE_URL"],
    api_key: ENV["WEAVIATE_API_KEY"],
    llm: :openai, # or :cohere
    llm_api_key: ENV["OPENAI_API_KEY"]
)

# You can instantiate any other supported vector search database:
client = Vectorsearch::Milvus.new(...) # `gem "milvus", "~> 0.9.0"`
client = Vectorsearch::Qdrant.new(...) # `gem"qdrant-ruby", "~> 0.9.0"`
client = Vectorsearch::Pinecone.new(...) # `gem "pinecone", "~> 0.1.6"`
```

```ruby
# Creating the default schema
client.create_default_schema
```

```ruby
# Store your documents in your vector search database
client.add_texts(
    texts: [
        "Begin by preheating your oven to 375°F (190°C). Prepare four boneless, skinless chicken breasts by cutting a pocket into the side of each breast, being careful not to cut all the way through. Season the chicken with salt and pepper to taste. In a large skillet, melt 2 tablespoons of unsalted butter over medium heat. Add 1 small diced onion and 2 minced garlic cloves, and cook until softened, about 3-4 minutes. Add 8 ounces of fresh spinach and cook until wilted, about 3 minutes. Remove the skillet from heat and let the mixture cool slightly.",
        "In a bowl, combine the spinach mixture with 4 ounces of softened cream cheese, 1/4 cup of grated Parmesan cheese, 1/4 cup of shredded mozzarella cheese, and 1/4 teaspoon of red pepper flakes. Mix until well combined. Stuff each chicken breast pocket with an equal amount of the spinach mixture. Seal the pocket with a toothpick if necessary. In the same skillet, heat 1 tablespoon of olive oil over medium-high heat. Add the stuffed chicken breasts and sear on each side for 3-4 minutes, or until golden brown."
    ]
)
```

```ruby
# Retrieve similar documents based on the query string passed in
client.similarity_search(
    query:,
    k:       # number of results to be retrieved
)
```
```ruby
# Retrieve similar documents based on the embedding passed in
client.similarity_search_by_vector(
    embedding:,
    k:       # number of results to be retrieved
)
```
```ruby
# Q&A-style querying based on the question passed in
client.ask(
    question:
)
```

### Using Standalone LLMs 🗣️

Add `gem "ruby-openai", "~> 4.0.0"` to your Gemfile.

#### OpenAI
```ruby
openai = LLM::OpenAI.new(api_key: ENV["OPENAI_API_KEY"])
```
```ruby
openai.embed(text: "foo bar")
```
```ruby
openai.complete(prompt: "What is the meaning of life?")
```

#### Cohere
Add `gem "cohere-ruby", "~> 0.9.3"` to your Gemfile.

```ruby
cohere = LLM::Cohere.new(api_key: ENV["COHERE_API_KEY"])
```
```ruby
cohere.embed(text: "foo bar")
```
```ruby
cohere.complete(prompt: "What is the meaning of life?")
```

#### HuggingFace
Add `gem "hugging-face", "~> 0.3.2"` to your Gemfile.

### Using Prompts 📋

#### Prompt Templates

Create a prompt with one input variable:

```ruby
prompt = Prompt::PromptTemplate.new(template: "Tell me a {adjective} joke.", input_variables: ["adjective"])
prompt.format(adjective: "funny") # "Tell me a funny joke."
```

Create a prompt with multiple input variables:

```ruby
prompt = Prompt::PromptTemplate.new(template: "Tell me a {adjective} joke about {content}.", input_variables: ["adjective", "content"])
prompt.format(adjective: "funny", content: "chickens") # "Tell me a funny joke about chickens."
```

Creating a PromptTemplate using just a prompt and no input_variables:

```ruby
prompt = Prompt::PromptTemplate.from_template("Tell me a {adjective} joke about {content}.")
prompt.input_variables # ["adjective", "content"]
prompt.format(adjective: "funny", content: "chickens") # "Tell me a funny joke about chickens."
```

Save prompt template to JSON file:

```ruby
prompt.save(file_path: "spec/fixtures/prompt/prompt_template.json")
```

Loading a new prompt template using a JSON file:

```ruby
prompt = Prompt.load_from_path(file_path: "spec/fixtures/prompt/prompt_template.json")
prompt.input_variables # ["adjective", "content"]
```

#### Few Shot Prompt Templates

Create a prompt with a few shot examples:

```ruby
prompt = Prompt::FewShotPromptTemplate.new(
  prefix: "Write antonyms for the following words.",
  suffix: "Input: {adjective}\nOutput:",
  example_prompt: Prompt::PromptTemplate.new(
    input_variables: ["input", "output"],
    template: "Input: {input}\nOutput: {output}"
  ),
  examples: [
    { "input": "happy", "output": "sad" },
    { "input": "tall", "output": "short" }
  ],
   input_variables: ["adjective"]
)

prompt.format(adjective: "good")

# Write antonyms for the following words.
#
# Input: happy
# Output: sad
#
# Input: tall
# Output: short
#
# Input: good
# Output:
```

Save prompt template to JSON file:

```ruby
prompt.save(file_path: "spec/fixtures/prompt/few_shot_prompt_template.json")
```

Loading a new prompt template using a JSON file:

```ruby
prompt = Prompt.load_from_path(file_path: "spec/fixtures/prompt/few_shot_prompt_template.json")
prompt.prefix # "Write antonyms for the following words."
```

### Using Agents 🤖
Agents are semi-autonomous bots that can respond to user questions and use available to them Tools to provide informed replies. They break down problems into series of steps and define Actions (and Action Inputs) along the way that are executed and fed back to them as additional information. Once an Agent decides that it has the Final Answer it responds with it.

#### Chain-of-Thought Agent

Add `gem "openai-ruby"`, `gem "eqn"`, and `gem "google_search_results"` to your Gemfile

```ruby
agent = Agent::ChainOfThoughtAgent.new(llm: :openai, llm_api_key: ENV["OPENAI_API_KEY"], tools: ['search', 'calculator'])

agent.tools
# => ["search", "calculator"]
```
```ruby
agent.run(question: "How many full soccer fields would be needed to cover the distance between NYC and DC in a straight line?")
#=> "Approximately 2,945 soccer fields would be needed to cover the distance between NYC and DC in a straight line."
```

#### Demo
![May-12-2023 13-09-13](https://github.com/andreibondarev/langchainrb/assets/541665/6bad4cd9-976c-420f-9cf9-b85bf84f7eaf)

![May-12-2023 13-07-45](https://github.com/andreibondarev/langchainrb/assets/541665/9aacdcc7-4225-4ea0-ab96-7ee48826eb9b)

#### Available Tools 🛠️

| Name     | Description          | Requirements         |
| -------- | :------------------: | :------------------: |
| "calculator" | Useful for getting the result of a math expression | |
| "search" | A wrapper around Google Search | `ENV["SERPAPI_API_KEY"]` (https://serpapi.com/manage-api-key)
| "wikipedia" | Calls Wikipedia API to retrieve the summary | |


## Logging

LangChain.rb uses standard logging mechanisms and defaults to `:debug` level. Most messages are at info level, but we will add debug or warn statements as needed.
To show all log messages:

```ruby
Langchain.logger.level = :info
```

## Development

After checking out the repo, run `bin/setup` to install dependencies. Then, run `rake spec` to run the tests. You can also run `bin/console` for an interactive prompt that will allow you to experiment.

To install this gem onto your local machine, run `bundle exec rake install`. To release a new version, update the version number in `version.rb`, and then run `bundle exec rake release`, which will create a git tag for the version, push git commits and the created tag, and push the `.gem` file to [rubygems.org](https://rubygems.org).

## Core Contributors
[<img style="border-radius:50%" alt="Andrei Bondarev" src="https://avatars.githubusercontent.com/u/541665?v=4" width="80" height="80" class="avatar">](https://github.com/andreibondarev)

## Honorary Contributors
[<img style="border-radius:50%" alt="Andrei Bondarev" src="https://avatars.githubusercontent.com/u/541665?v=4" width="80" height="80" class="avatar">](https://github.com/andreibondarev)
[<img style="border-radius:50%" alt="Rafael Figueiredo" src="https://avatars.githubusercontent.com/u/35845775?v=4" width="80" height="80" class="avatar">](https://github.com/rafaelqfigueiredo)

(Criteria of becoming an Honorary Contributor or Core Contributor is pending...)

## Contributing

Bug reports and pull requests are welcome on GitHub at https://github.com/andreibondarev/langchain.

## License

The gem is available as open source under the terms of the [MIT License](https://opensource.org/licenses/MIT).
