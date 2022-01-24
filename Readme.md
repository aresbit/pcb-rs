# Pcb-rs

Currently this is a scratchpad for writing notes and stuff

## Trisatable pins

In real pcb, the individual pins can only transfer voltages, (thus bits), and are connected to each other. Here we allow pins to have more complex data types, at expense of possible runtime panics. (as technically the types are resolved at file level, two files can have two diff types which have same name, and while generating macros, types are not resoled, so we do not have enough information until runtime if both types are same or not. )

If we allow only single connection per pin, it can not only get complicated to implement chips which connect to multiple devices, but also it might not be possible to establish shared connection at all. For eg : in a particular system RAM must be connected to both CPU and a DMA module. Now if we don't allow multiple connections, we cannot connect data pins of RAM to both CPU and DMA. That means either only one can access the RAM, or we have to add a layer of indirection between RAM and other components such that CPU and DMA will request to this component and the pins of this component will be connect to RAM. Even then, in that component we cannot connect the data pin granting pin of that indirection chip to both, due to the same issue. That means we will need one pin for each connected component, and, some priority based method to tie brake if multiple components request access to data pin. This can turn quite inefficient as the number of components to be connected grows.

In real world, such issue is solved by two methods : see [this](https://www.microchip.com/forums/m641935.aspx) for a good explanation. In this library, we use tristating. That way ideally only one of the connected chip will have a valid output (High or Low) and others will be in High-Z mode, where essentially that pin acts as if it is not connected at all. Although in case multiple chips connected to same pin do go in non-high-z state at the same point, it will cause issues, potentially burning of real chips. Also see [this](https://en.wikipedia.org/wiki/Three-state_logic).

In case of this library, we use rust std::option::Option to indicate that a pin is tristatable, and multiple pins are allowed to connect to same pin only if all are tristatable. The case of multiple tristatable pins have Some(\_) at the same time, this is equivalent to multiple pins going high/low at the same time, and thus the code will panic at runtime, equivalent to the chip burning.

They way this is implemented is not the best or elegant way, but that was the only feasible way I could find.
