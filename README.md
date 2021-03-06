# Quantities

A C++ template meta-programming library for enforcing dimensional consistency at compile time. Inspired by Boost::units, partly written as an exercise for myself but also to handle mixed types nicely.

## Use
### Dimensions

**Namespace: `dims`**

Physical quantities can be wrapped in a `quantity` class which enforces that calculations involve only the correct physical dimensions. For example `quanity<mass> m = 2.0;` would create a variable `m` which has dimensions of mass. If we were to multiply this by an acceleration to get a force we would only be able to store it in a variable of type `quantity<force>`

    quantity<mass> m = 2.0;
    quantity<acceleration> a = 3.0;
    quantity<force> f = m*a; // okay
    quantity<work> w = m*a; // causes compiler error

In this way accidental attempts to store variables in the wrong types are caught at compile time.

We can also save ourselves some typing through the use of the `auto` keyword. The last line in the example above would become `auto f = m*a;` and the compiler automatically deduces the correct type.

The `quantity` template class works nicely with various different types performing the correct multiplication and division operations on these. For example imagine we have a vector class called `vec3`. It has an overloaded multiplication operator `vec3 vec3::operator(double d)` representing scaling the vector and a constructor `vec3(double x, double y, double z)`. The the following code is perfectly valid

    quantity<mass> m = 2.0;
    quantity<acceleration,vec3> a(3.0,2.0,1.0) // the constructor for quantity<DIM,vec3> forwards the arguments to the constructor for vec3
    auto f = m*a;

Yielding an object `f` which has type `quantity<force,vec3>`.

### Units

**Namespace: `units`**

Another key ability is the ability to handle units nicely. As with Boost::units Quantities provides this capability; here through the use of the `unit` template class. Note, however, that the units part is not as neat as the dimensions part and I would recommend using Boost::units.

The `unit` class is templated on a dimension (same as in above section), a system of units and a type; below is an example

    unit<mass,si_system> m = 2.0*kilogram;

Notice that the type defaults to `double` as with `quantity`. The unit system is merely a `static_list` containing types specifying which base unit to use for each fundamental dimension. So the `si_system` is defined as `static_list<si::kg_t,si::m_t,si::s_t>` to represent kilograms, meters and seconds. This means that users can create their own systems of units if they so desire. It is even possible to create new fundamental units by defining custom types and converters *[see section Converters]*. As with Boost::units quantities are automatically converted to be in the correct units.

    unit<length,si_system> l = 4.0*meter;
    unit<area,si_system> a = l*(1.0*cm);
    cout << a << endl; // should be 0.04

In the example above `1.0*cm` is automatically converted to meters when it is multiplied by `l` since `l` is using SI units.
