# google-cloud-language

[Google Cloud Natural Language API](https://cloud.google.com/natural-language/) ([docs](https://cloud.google.com/natural-language/docs)) reveals the structure and meaning of text by offering powerful machine learning models in an easy to use REST API. You can use it to extract information about people, places, events and much more, mentioned in text documents, news articles or blog posts. You can use it to understand sentiment about your product on social media or parse intent from customer conversations happening in a call center or a messaging app. You can analyze text uploaded in your request or integrate with your document storage on Google Cloud Storage.

The Cloud Natural Language API currently supports English, Spanish, and Japanese for sentiment analysis, entity analysis, and syntax analysis.

- [google-cloud-language API documentation](http://googlecloudplatform.github.io/google-cloud-ruby/#/docs/google-cloud-language/latest)
- [google-cloud-language on RubyGems](https://rubygems.org/gems/google-cloud-language)
- [Google Cloud Natural Language API documentation](https://cloud.google.com/natural-language/docs)

## Quick Start

```sh
$ gem install google-cloud-language
```

## Authentication

This library uses Service Account credentials to connect to Google Cloud services. When running on Compute Engine the credentials will be discovered automatically. When running on other environments the Service Account credentials can be specified by providing the path to the JSON file, or the JSON itself, in environment variables.

Instructions and configuration options are covered in the [Authentication Guide](https://googlecloudplatform.github.io/google-cloud-ruby/#/docs/google-cloud-language/guides/authentication).

## Example

```ruby
require "google/cloud/language"

language = Google::Cloud::Language.new

content = "Star Wars is a great movie. The Death Star is fearsome."
document = language.document content
annotation = document.annotate

annotation.entities.count #=> 3
annotation.sentiment.score #=> 0.10000000149011612
annotation.sentiment.magnitude #=> 1.100000023841858
annotation.sentences.count #=> 2
annotation.tokens.count #=> 13
```

## Supported Ruby Versions

This library is supported on Ruby 2.0+.

## Versioning

This library follows [Semantic Versioning](http://semver.org/).

It is currently in major version zero (0.y.z), which means that anything may change at any time and the public API should not be considered stable.

### Lower-level (GAPIC) API versioning

The lower-level API support contained within this package is versioned separately to match its corresponding Google Cloud Platform service API version.

*Please note that in lower-level API support in this package, version `v1beta2` comes after version `v1`. The API version `v1beta2` contains new features that were not released in `v1`.*

## Contributing

Contributions to this library are always welcome and highly encouraged.

See the [Contributing Guide](https://googlecloudplatform.github.io/google-cloud-ruby/#/docs/guides/contributing) for more information on how to get started.

Please note that this project is released with a Contributor Code of Conduct. By participating in this project you agree to abide by its terms. See [Code of Conduct](../CODE_OF_CONDUCT.md) for more information.

## License

This library is licensed under Apache 2.0. Full license text is available in [LICENSE](LICENSE).

## Support

Please [report bugs at the project on Github](https://github.com/GoogleCloudPlatform/google-cloud-ruby/issues).
Don't hesitate to [ask questions](http://stackoverflow.com/questions/tagged/google-cloud-platform+ruby) about the client or APIs on [StackOverflow](http://stackoverflow.com).
