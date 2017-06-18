---
layout: post
title: iOS BDD/TDD using the Presenter pattern
---

Troubled with writing unit tests? TDD? It's not unusual to find iOS projects that don't enforce the idea that much. If you are in need to increase code quality, this can hopefully help you.

_Test driven development_ is a practice that will help you test more and write better code, but more than that, it helps you code without fears, be it new features or big refactorings.
The _Presenter_ pattern is something that can help you bridge the gap and get started in iOS TDD.

We will talk a bit about both, how to integrate them and then write a small example.

---

## A little background on TDD and BDD

### TDD

The idea of __Test Driven Development__ (for short, TDD) is pretty simple. Before writing a single line of code, you write the test that covers the feature you want to implement. The test will fail, and based on those failures you will write code that will help those failures turn into success. This then turns into an iterative process of:
* Covering features with unit tests
* Implementing code that will help you pass them
* Refactor to keep it clean. 

Simple, right?

### BDD

__Behavior Driven Development__ is a development practice that further extends TDD. Its main focus is to describe feature scenarios verbosely (in terms of [GiveWhenThen](https://dannorth.net/introducing-bdd/)) based on a program's behavior and then write the program to satisfy that described behavior. It can be mixed with the concept of TDD to close the gap between user stories and the code that implements them.

### Mocks and Stubs

Mocks and Stubs will be used in this article. If you aren't familiar with these concepts, I suggest you take a look at [this article](https://martinfowler.com/articles/mocksArentStubs.html#TheDifferenceBetweenMocksAndStubs).

---

## Unit Testing UI? Presenter to the rescue

A common problem that is faced in unit testing is that many iOS (and mobile) app features tend to focus on the UI. This can be solved with UI testing libraries or Xcode's UI testing tool. However, a lot of the underlying UI and UX logic may be harder to test that way. A simple way to achieve testing of the logical layer is by using the Presenter pattern.

### The Presenter

The __Presenter__ pattern can be found in architectural patterns like MVP or VIPER (You can read about them in [this very good post](https://medium.com/ios-os-x-development/ios-architecture-patterns-ecba4c38de52)). The main idea of the Presenter is to separate UI logic from the View layer. The View layer (in this case the View Controller) being the component that interacts directly with UI components (like UIButtons, UITextfields, etc.). 

At the end, the Presenter receives interactions from the UI and decides how the UI should behave (making the actual changes in the UI is responsibility of the View). This is pretty much a high level description of how the UI should function, which makes it easier to translate user story scenarios into our code. 

<a href="{{ site.baseurl }}/assets/bdd-presenter-tests-graph-black.png"><img style="width:80%; padding-top:30px; margin:auto" src="{{ site.baseurl }}/assets/bdd-presenter-tests-graph-black.png"/></a>

The Presenter can serve as a good _starting point_ for translating features into unit test cases. From there, the rest of tests and dependencies (models, interactors, data providers, helpers, etc.) to complete the feature become clear.

---

## Let's get this rolling - Practical example

For this example, we will do a BDD implementation for a feature to be designed with MVP.
To help us write BDD unit tests, we will use [Quick](https://github.com/Quick/Quick).

### Step 1: Read your acceptances / user story

The usual flow in BDD is that user stories are written describing behavior. That's pretty straightforward and can be ported directly to your unit tests. 

We will analyze a case with a regular user story and see how it transitions into BDD style syntax and finally end up as presenter tests.

Given the following story:

> As a user, I want to search for beers that go well with desired food, so that I can view which beers go better with the food I have.
>
> Acceptance:
- A pageable list of beer images is returned after searching for pairing food.
- The search button will be enabled when there is one or more valid characters searched. Valid characters include only letters (a-zA-Z) and whitespace.
- If the search includes invalid characters, the search button will be disabled.
- Entering more than 3 words should instead display an alert with the following message: "Don't go too crazy with the food!". For this case, we won't consider validations for misplaced whitespaces (like at the start of the string, double spaces, etc.).
- If an error occurs while searching, a alert is displayed with the following error: "There was a problem fetching your beers. Please try again later."

### Step 2: Translate into expected behavior and Presenter tests

We then extract the expected behavior from the user story to a Given When Then or similar syntax into a list of scenarios or test cases.

1. It should display a list of beer images *WHEN* food is searched *GIVEN* that it contains 3 or less words
2. It should display a message *WHEN* food is searched *GIVEN* that it contains more than 3 words
3. It should display an error message *WHEN* food is searched *GIVEN* that there is an error while doing a valid search
4. It should disable the search action *WHEN* starting
5. It should enable the search action *WHEN* entering search text *GIVEN* only valid characters are entered
6. It should disable the search action *WHEN* entering search text *GIVEN* invalid characters are entered

These kind of behaviors translate very well into presenter tests. With any BDD framework or format, their verbosity can be maintained as well. The skeleton for our presenter test cases in Quick should look something like this:

```swift
class BeerSearchTests: QuickSpec {
    override func spec() {
        describe("presenter") {
            describe("search") {
                context("given search text containts 3 or less words") {
                    it("should display a list of beer images") {
                    }
                }

                context("given search text containts more than 3 words") {
                    it ("should display a message") {
                    }
                }

                context("given valid search error") {
                    it ("should display a error message") {
                    }
                }
            }

            describe("start") {
                it("should disable the search action") {
                }
            }

            describe("set search text") {
                context("given only valid characters are entered") {
                    it("should enable the search action") {
                    }
                }

                context("given invalid characters are entered") {
                    it("should disable the search action") {
                    }
                }
            }
        }
    }
}
```

### Step 3: Implementing unit tests for dependencies and business logic components

**The amount of code that will be shown could be quite substancial. To make it easier to follow, you can check the project source at Github.**

For this example, we will focus on the first 2 scenarios. Before moving on to writing the presenter tests, we need the logical/computational portion and dependencies of this scenario implemented first. Let's create a SearchTermTests class that will contain the tests we need for this scenario.

```swift
class SearchTermTests: QuickSpec {
    override func spec() {
        describe("search term") {
            describe("has valid word count") {
                context("given text that containts 3 or less words") {
                    it("should return true") {
                        let searchTerm = SearchTerm(text: "pizza")
                        let result = searchTerm.hasValidWordCount()
                        expect(result).to(beTruthy())
                    }
                }

                context("given text that containts more than 3 words") {
                }
            }
        }
    }
}
```

You will notice that this test does not compile. That's because the SearchTerm type does not even exist! So let's go and implement it until the compiler does not complain.

```swift
struct SearchTerm {
    let text: String
    
    func hasValidWordCount() -> Bool {
        return true
    }
}
```

With this, the test will compile. Plus, since we are just returning true, the test will pass as well. While this implementation is not correct, it is good enough to pass our test. We don't need to worry about how the final implementation of our method should be like, as our tests will help us shape it correctly. Let's implement the other test for the SearchTerm type now.

```swift
context("given text that containts more than 3 words") {
    it ("should return false") {
        let searchTerm = SearchTerm(text: "cheese hawaiian pepperoni pizza")
        let result = searchTerm.hasValidWordCount()
        expect(result).to(beFalsy())
    }
}
```

Not surprisingly, this test will fail. In order to make it pass, let's modify our *hasValidWordCount* function.

```swift
func hasValidWordCount() -> Bool {
    let components = text.components(separatedBy: " ")
    if components.count < 4 {
        return true
    } else {
        return false
    }
}
```

With this, we've managed to pass both tests. The other dependency we have is the service class that will work as an adapter to our HTTP client. In this case, we will just work with a stubbed class since we don't want to go all the way and have actual HTTP requests in our unit tests. While we are leaving the adapter out for now, we can later test it as part of our integration tests (if you can afford having a test or fake backend) or simply stubbing out the HTTP client.

### Step 4: Implementing unit tests for the presenter

As we previously outlined, we will take a look at the unit tests for the first 2 scenarios. For the first scenario, the output we want to test is:

> should display a list of beer images

Since the output happens in the form of an interaction, we will need to create a mock of the view in order to test that specific interaction. The unit test we write would look like this:

```swift
describe("presenter") {
    let searchService = SearchServiceStub()
    let view = SearchViewMock()
    let presenter = SearchPresenter(view: view, searchService: searchService)
    ...
    describe("search") {
        context("given search text containts 3 or less words") {
            it("should display a list of beer images") {
                // Given
                searchService.returnBeerList = [
                    Beer(imageURL: "https://testurl1.com"),
                    Beer(imageURL: "https://testurl2.com")
                ]
                
                // When
                presenter.search(text: "pizza")
                
                // Then
                let expectedImages = [
                    "https://testurl1.com",
                    "https://testurl2.com"
                ]
                expect(view.calledDisplayBeers).toEventually(equal(expectedImages))
            }
        }
    ...
```

It's no surprise that from the previously written unit test, the compiler will be filing a mountain of complains. That's obvious since most of those types and methods don't exist yet. Our first mission is to make the unit test compile. For our first compile error, let's create the search service stub.

```swift
struct Beer {
    let imageURL: String
}

protocol SearchService {
    func searchBeers(foodText: String, completion:([Beer]) -> ())
}

class SearchServiceStub: SearchService {
    var returnBeerList: [Beer]?
    
    func searchBeers(foodText: String, completion: ([Beer]) -> ()) {
        if let returnBeerList = returnBeerList {
            completion(returnBeerList)
        }
    }
}
```

Then, let's proceed to create the view protocol and mock.

```swift
protocol SearchContractView {
    func displayBeerImages(_ imageUrlList:[String])
}

class SearchViewMock: SearchContractView {
    var calledDisplayBeers: [String]?
    
    func displayBeerImages(_ imageUrlList: [String]) {
        calledDisplayBeers = imageUrlList
    }
}
```

And finally, proceed to create the presenter for the test to pass.

```swift
protocol SearchContractPresenter {
    func search(text: String)
}

class SearchPresenter: SearchContractPresenter {
    weak var view: SearchContractView?
    let searchService: SearchService
    
    init(view: SearchContractView, searchService: SearchService) {
        self.view = view
        self.searchService = searchService
    }
    
    func search(text: String) {
        guard let view = view else { return }

        searchService.searchBeers(foodText: text) { beerList in
        view.displayBeerImages(beerList.map({ beer -> String in
            return beer.imageURL
        }))
    }
}
```

If the _weak_ modifier on the view caught your intention, it is done so to avoid a retain cycle as normally the class implementing the view would own the presenter.

### Step 5: Finishing the second and third behaviors

Finally complete the rest of tests. The other two tests would look like:

```swift
context("given search text containts more than 3 words") {
    it ("should display a message") {
        // When
        invalidSearch()
        
        // Then
        let expectedMessage = "Don't go too crazy with the food!"
        expect(view.calledDisplayMessage).toEventually(equal(expectedMessage))
    }
}

context("given valid search error") {
    it ("should display a error message") {
        // Given
        searchService.returnBeerError = SearchError.genericError
        
        // When
        validSearch()
        
        // Then
        let expectedMessage = "There was a problem fetching your beers. Please try again later."
        expect(view.calledDisplayErrorMessage).toEventually(equal(expectedMessage))
    }
}
```

Which will finally make our presenter search function to look like this:

```swift
func search(text: String) {
    guard let view = view else { return }

    let searchTerm = SearchTerm(text: text)
    if searchTerm.hasValidWordCount() {
        searchService.searchBeers(foodText: text, completion: { (beerList, error) in
            if error != nil {
                view.displayErrorMessage("There was a problem fetching your beers. Please try again later.")
            } else {
                view.displayBeerImages(beerList.map({ beer -> String in
                    return beer.imageURL
                }))
            }
        })
    } else {
        view.displayMessage("Don't go too crazy with the food!")
    }
}
```

We ended up creating only the necessary fake objects (test doubles, etc.) to intentionally verify the interaction of the presenter and the view, and to prevent hitting a real backend. As the lack of reflection in Swift difficults the integration of mocking frameworks, there are clear benefits in keeping fake objects to a minimum. We won't go into the specifics of how much you should mock, as that falls more into your TDD stance and the strategy your team is willing to undertake.

---

## Let's wrap up

Well, this ended up being a pretty big article. That concludes the presenter unit tests and implementation for the first three proposed behaviors. The rest is left as a small exercise. If you didn't see the reference above, you can check the project source here.

In this occasion, we mostly did a walkthrough in TDD, while emphasizing the synergy between BDD and patterns like MVP or VIPER. It is not to say that a similar approach can't be taken while preparing UI tests. However, exercising this path can be another asset to maintain a more fluid workflow, specially if your project benefits from these architectural patterns. 
