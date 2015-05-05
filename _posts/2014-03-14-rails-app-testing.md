---
layout: post
title: "Rails App Testing"
description: "Standards, Tips, and Performance for testing your Rails app"
category:
tags: [rspec, rails, api]
---
{% include JB/setup %}


__Disclaimer:__ This is just my opinion on rails testing. It's no way a *"must follow guide for testing"* __Also this blog post is a work progress.__

#Api Testing
A good rule of thumb for writing tests for your api is: You should test every thing in your api documentation to make sure your endpoints, accepted params, and response body does what it says it does. With that being said, this makes sure your api documentation is tested and your clients can trust it. The clients depending on your api shouldn’t have to tell you when your api is broken because you’ve made changes to it. Your tests should let you know this ahead of time.

##Things you should test when writing API tests
###The serializer/response body
This way you will know exactly what the serializer is returning and you can be confident when you have to make changes to your models or serializers that it won’t break the api.  

At minimum you should be testing the keys. That way you know what keys you’re returning with the api.

You can write and test your response body in your controller tests or in your serializer tests.

#####Examples

If the documentation says that `GET /user/:id` will return:

    {
       first_name: string,
       last_name: string
    }
Then you should at least have a test that does something along the lines of:

    describe '#GET user/:id' do
       expected = {
          first_name: user.first_name,
          last_name: user.last_name,
       }.to_json
       get :show, id: 1
       response.body.should eq expected
    end

If you want to test the serializer you can do something like this:

These should go in the `spec/serializers` folder.

    describe UserSerializer do  
      it "should return the right attributes for a user" do
        serializer = UserSerializer.new User.new(id: 1, first_name: 'first', last_name: 'last')
        expect(serializer.to_json).to eql('{"user":{"id":1,"first_name":"first", "last_name":"last"}}')  
      end
    end

The above can be harder to write and but it is more verbose. If you don't like that option you can write controller tests and use the .all? block to test the keys of the json.

`api/users_controller_spec.rb`

    describe Api::UsersControllers do
      let!(:user){ Fabricate :user }
      before do
        get api_users_path
      end
      it 'should have the correct attributes' do
        get :show, id: 1
        json = JSON.parse(response.body)  
        expect{
          user.attributes.all? do |key|
            json.has_key(key)
          end
        }.to be
      end
    end

The code above checks to make sure that all the keys for the model are represented in the response body. If you need to add more attributes you can alway just give the .all? block a hash with all the keys that should be in there. This will not only check if all your keys are there but will fail if you add or remove a key from your serializer.

good sources and gems for testing can be found here:

[https://github.com/ahawkins/active_model_serializers-matchers](https://github.com/ahawkins/active_model_serializers-matchers)

[http://dhartweg.roon.io/rspec-testing-for-a-json-api](http://dhartweg.roon.io/rspec-testing-for-a-json-api)

###Endpoints

You should check accepted params for the endpi. required and optional

These are easy test to write just because you're just checking the model validations.

#####Examples
`models/user.rb`

    class User < ActiveRecord::Base
      validates :first_name, :last_name, presence
    end

You should always tests your models validation code. This way you know that you’re validations are working properly.

#Model tests

##What should you be testing

   * Model validations

   * Instance and class methods

###Model Validations
You should always start with a valid fixture.

`user_fabricator.rb`

      Fabricator(:user) do
        first_name { sequence(:first_name) { |n| "first#{n}" } }
        first_name { sequence(:last_name) { |n| "last#{n}" } }
      end

`user_spec.rb`

    describe User do
         let(:user){ Fabricate.build :user }
         subject { user }
         it{ should be_valid }
    end

This makes sure you have all the correct data in your fixture. It kinda tests your model validations as well but I still like to be more verbose about this. This is why I've found something very easy to do this as well. You can use [shoulda-matchers](https://github.com/thoughtbot/shoulda-matchers) to test your model validations. I'll write 2 examples below, One with shoulda and one without.

*Without shoulda-matchers*

      describe User do
        describe 'must have first name' do
          before { subject.first_name = nil }
          it{ should be_invalid }
        end
      end

*With shoulda-mathchers*

      describe User do
        it { should validate_presence_of :first_name }
      end


###Instance and class methods

You should be testing you model instance and class methods as well. This is important because I've seen a lot of times were people make instance methods for some view code and then someone comes along and changes it. Then this breaks ever where this instance code was used and no one knows about until it goes into production. Remember tests are to protect you and others from changing something that is being used somewhere else in code base. I'll start out with something basic for testing instance methods


`user.rb`

      class User < ActiveRecord::Base
        def to_s
          [first_name, last_name].join
        end
      end

`user_spec.rb`

      describe User do
        its(:to_s){ should eq [subject.first_name, subject.last_name].join }
      end
