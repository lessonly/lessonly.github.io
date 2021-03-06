---
layout: styleguide
title: Feature Flags Style Guide
main: true
---

### Feature Flags

#### Create Uniquely-Named Feature Flags

Even when features live in different policies, they must all have unique names.

#### Add Feature Policy Mapping to `AccessControl::Features::FeaturePolicies`

The `FEATURE_POLICY_MAPPINGS` object maps feature names to the policies in which they can be found, which is needed in some areas of the application. Features are added to this mapping alphabetically.

#### Add Translation Key for New Features

```ruby
# Path to feature-related translation keys in config/locales/en.yml
en:
  admin:
    company_details:
      features:
        feature_name: 'Classification: Name of Feature'
# Without this addition, feature_name will be shown in Active Admin as "Feature Name".
```

#### Favor Calling Feature Checks & Toggles Directly on Policy

```ruby
# Old Way, Avoid
company.has_feature?(:feature_name)
company.enable_feature!(:feature_name)
company.disable_feature!(:feature_name)

# New Way, Recommended
# This way is favored because we can perform feature checks on more types of objects.
# The "old way" only allowed for company, now we can also pass in users & groups.
AccessControl::Features::ExamplePolicy.new(user).feature_name?
AccessControl::Features::ExamplePolicy.new(company).grant!(:feature_name)
AccessControl::Features::ExamplePolicy.new(company).revoke!(:feature_name)
```

#### Favor Checking Features on the Most Specific Level

```ruby
# Not recommended, very broad
AccessControl::Features::ExamplePolicy.new(company).feature_name?

# Recommended!
# This goes for Policies & Permissions as a whole.
AccessControl::Features::ExamplePolicy.new(user).feature_name?
```

#### When Writing Unit Tests, Prefer Stubbing Feature Flag Checks

This does not apply to integration tests!
[See Testing Style Guide](https://about.lessonly.engineering/styleguide/testing/#avoid-stubbing-and-mocking-in-integration-tests) for more information on why.

```ruby
# Fine, But Not Ideal
before do
  AccessControl::Features::ExamplePolicy.new(company).grant!(:feature_name)
end

# Recommended
# This can improve performance and reduce potential issues.
before do
  example_policy_double = instance_double(AccessControl::Features::ExamplePolicy)
  expect(example_policy_double).to receive(:feature_flag_name).and_return(true)

  expect(AccessControl::Features::ExamplePolicy).to receive(:new).with(company).and_return(example_policy_double)
end
```

#### Front End Implementation

There is documentation in the React Style Guide which outlines how the `FeatureFlag` component can be used to do feature checks on React pages.

[For more information, please visit that section of the Style Guide.](https://about.lessonly.engineering/styleguide/react/#featureflag-interface--in-transition---frontend-experiment)
