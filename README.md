# metamorph

A Clojure library designed to providing pipelining operations.

### Pipeline operation

Pipeline operation is a function which accepts context as a map and returns possibly modified context map.

#### Context

Context is just a map where pipeline information is stored. There are three reserved keys which are supposed to help organize a pipeline:

* `:metamorph/data` - object which is subject to change and where the main data is stored. It can be anything: dataset, tensor, object, whatever you want
* `:metamorph/id` - unique operation number which is injected to the context just before pipeline operation is called. This way pipeline operation have some identity which can be used to store and restore private data in the context.
* `:metamorph/mode` - additional context information which can be used to determine pipeline phase. It can be added explicitely during pipeline creation.
  Different pipeline functions can work together, if they agree on a common set of modes and act accordingly depending on the mode.
  The main use case for this are pipelines which include a statistical model in some form. In here the model either gets fitted on the data (= learns form data) or it gets applied to data. For this common use case we define two standard modes, namely:
    * `:metamorph/fit`  - While the pipeline has this mode, any function in the pipeline should fit its model it from the data , this is as well called "train". It should write as well the fitted model to the key in `:metamorph/id` so, that on the next pipeline run in mode `transform` it can be used
    * `:metamorph/transform` - While the pipeline is in this mode, the fitted model should be read from the key in `:metamorph/id` and applied to the data:
 In machine learning terminology, these tow modes are typically called train and predict. In metamorph we use the fit/transform terms as well for machine learning pipelines.
 
### Compliant functions
All the steps of a metamorph pipeline are function which need to follow the following conventions, in order to work well together:

* Be usual Clojure functions which have at least one parameter, and this first parameter need to be a map. This map can contain any key.
* The value of a compliant function, need to be a a function which value is a map by itself. The function is allowed to add any keys with any value to the map, but should normally not remove any key. 
* The object under `:metamorph/data` is considered to be the main data object, which nearly all functions will interact with. A functions which only interacts with this main data object, need nevertheless return  the whole context map.
* Each function which reads or  writes specific keys to the pipeline context, should document this and use namespaced keys to avoid conflicts

### Metamorph compliant libraries
The existing clojure libraries `tablecloth`,`tech.ml.dataset` and `tech.ml` will be extended to make methamorph compliant functions available.

### Similar concept in sklearn pipelines
The `metamorph` concept is similar to the `pipeline` concept of sklearn, which allows as well to run a give pipeline in `fit` and `transform`.
But metamorph allows to combine models with arbitrary transform functions, which don't need to be models.


### Two types of functions in pipeline

We foresee that mainly 2 types of functions get added to a pipeline.

1. `State independend functions:` They only manipulate the main data object, and will ignore all other information in contexts.
  Neither will they use `mode` or the `id` in the context map.
2. `State dependend functions`: These functions will behave different depending on the mode and will likely store data in the context map, which can be used by the same function in an other mode or by other functions later in the pipeline.

### Pipelines can be constrcucted from function or as pure data
Metamorph pipelines can be either constructed from a sequence of function calls or declarative as a sequence of maps.

Both rely on the same functions.

This should allow advanced use cases, like the **generation** of pipelines,
which gives large flexibility for hyper parameter tuning in machine learning.
### Advantages of the metamorph concept

* A complete (machine learning) pipeline becomes self contained. All information (data and "state" of models) is inside the pipeline context
* All steps of the pipeline are pure functions, which outcome depends only on its context map parameter and eventual options
* It unifies the data pipeleine idea ot `tablecloth` with the concept of fitted models
* Only pure Clojure data structures
* No dependency to any concrete data manipulation library, but all can be integrated easely
 
#### Creating a pipeline

To create a pipeline you can use two functions:

* `pipeline` to make a pipeline function out of pipeline operators
* `->pipeline` as an above, but using declarations

## Usage

TODO!

## License

Copyright © 2021 Scicloj

This program and the accompanying materials are made available under the
terms of the Eclipse Public License 2.0 which is available at
http://www.eclipse.org/legal/epl-2.0.

This Source Code may also be made available under the following Secondary
Licenses when the conditions for such availability set forth in the Eclipse
Public License, v. 2.0 are satisfied: GNU General Public License as published by
the Free Software Foundation, either version 2 of the License, or (at your
option) any later version, with the GNU Classpath Exception which is available
at https://www.gnu.org/software/classpath/license.html.
