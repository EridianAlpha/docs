# Library

* Can't have state variables.
* Can't send ETH.
* All functions must be internal.
* Library functions can be called directly if they do not modify the state.
  * That means pure or view functions only can be called from outside the library.
* Library can not be destroyed as it is assumed to be stateless.
* A Library cannot inherit any element.
* A Library cannot be inherited.
