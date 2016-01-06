Finite size
-----------

This section introduces the concept of shapes with classes :class:`.Polygon` and
:class:`.FreeformShape` which are used to model systems of finite size. The sparse
eigensolver :func:`.arpack` is also introduced as a good tool for exactly solving
larger Hamiltonian matrices.


Example
*******

Note the high intensity states on the inner and outer edges of the graphene ring.

:download:`Source code </tutorial/finite_example.py>`

.. plot:: tutorial/finite_example.py
    :include-source:

Polygon
*******

Finite sized systems require a shape. The simplest way to create a 2D shape is with the
:class:`.Polygon` class. For example, a simple rectangle:

.. plot::
    :context: reset

    def rectangle(width, height):
        x0 = width / 2
        y0 = height / 2
        return pb.Polygon([[x0, y0], [x0, -y0], [-x0, -y0], [-x0, y0]])

    shape = rectangle(1.6, 1.2)
    shape.plot()

A :class:`.Polygon` is initialized with a list of vertices which should be given in clockwise or
counterclockwise order. When added to a :class:`.Model` the lattice will expand to fill the shape.

.. plot::
    :context: close-figs

    from pybinding.repository import graphene

    model = pb.Model(
        graphene.lattice.monolayer(),
        rectangle(width=1.6, height=1.2)
    )
    model.system.plot()

To help visualize the shape and the expanded lattice, the polygon outline can be plotted on top
of the system by calling both plot methods one after another.

.. plot::
    :context: close-figs

    def trapezoid(a, b, h):
        return pb.Polygon([[-a/2, 0], [-b/2, h], [b/2, h], [a/2, 0]])

    model = pb.Model(
        graphene.lattice.monolayer(),
        trapezoid(a=3.2, b=1.4, h=1.5)
    )
    model.system.plot()
    model.shape.plot()


Freeform shape
**************

Unlike a :class:`.Polygon` which is defined by a list of vertices, a :class:`.FreeformShape` is
defined by a `contains` function which determines if a lattice site is inside the desired shape.

.. plot::
    :context: close-figs

    def circle(radius):
        def contains(x, y, z):
            return np.sqrt(x**2 + y**2) < radius

        return pb.FreeformShape(contains, width=[2 * radius, 2 * radius])

    model = pb.Model(
        graphene.lattice.monolayer(),
        circle(radius=2.5)
    )
    model.system.plot()

The `width` parameter of :class:`.FreeformShape` specifies the bounding box width. Only sites
inside the bounding box will be considered for the shape. It's like carving a sculpture from a
block of stone. The bounding box can be thought of as the stone block, while the `contains`
function is the carving tool that can give the shape fine detail.

As with :class:`.Polygon`, we can visualize the shape with the :meth:`.FreeformShape.plot` method.

.. plot::
    :context: close-figs

    def ring(inner_radius, outer_radius):
        def contains(x, y, z):
            r = np.sqrt(x**2 + y**2)
            return np.logical_and(inner_radius < r, r < outer_radius)

        return pb.FreeformShape(contains, width=[2 * outer_radius, 2 * outer_radius])

    shape = ring(inner_radius=1.4, outer_radius=2)
    shape.plot()

The shaded area indicates the shape as determined by the `contains` function. Creating a model
will cause the lattice to fill in the shape.

.. plot::
    :context: close-figs

    model = pb.Model(
        graphene.lattice.monolayer(),
        ring(inner_radius=1.4, outer_radius=2)
    )
    model.system.plot()
    model.shape.plot()

Note that the `ring` example uses `np.logical_and` instead of the plain `and` keyword. This is
because the `x, y, z` positions are not given as scalar numbers but as `numpy` arrays::

    >>> x = np.array([7, 2, 3, 5, 1])
    >>> x < 5
    [False, True, True, False, True]
    >>> 2 < x and x < 5
    ValueError: ...
    >>> np.logical_and(2 < x, x < 5)
    [False, False, True, False, False]

The `and` keyword can only operate on scalar values, but `np.logical_and` can consider arrays.
Likewise, `math.sqrt` does not work with arrays, but `np.sqrt` does.


Spatial LDOS
************

Now that we have a ring structure, we can exactly diagonalize its `model.hamiltonian` using a
:class:`.Solver`. We previously used the :func:`.lapack` solver to find all the eigenvalues and
eigenvectors, but this is not efficient for bigger systems. The sparse :func:`.arpack` solver can
calculate a targeted subset of the eigenvalues, which is usually desired and much faster. In this
case, we are interested only in the 20 lowest energy states.

.. plot::
    :context: close-figs

    model = pb.Model(
        graphene.lattice.monolayer(),
        ring(inner_radius=1.4, outer_radius=2)
    )
    solver = pb.solver.arpack(model, num_eigenvalues=20)

    ldos = solver.calc_spatial_ldos(energy=0, broadening=0.05)  # eV
    ldos.plot_structure(site_radius=(0.03, 0.12))

The convenient :meth:`.Solver.calc_spatial_ldos` method calculates the local density of states
(LDOS) at every site for the given energy with a Gaussian broadening. The returned object is a
:class:`.StructureMap` which holds the LDOS data. The :meth:`.StructureMap.plot_structure` method
will produce a figure similar to :meth:`.System.plot`, but with a colormap indicating the LDOS
value at each lattice site. In addition, the `site_radius` argument specifies a range of sizes
which will cause the low intensity sites to appear as small circles while high intensity ones
become large. The states with a high LDOS are clearly visible on the outer and inner edges of the
graphene ring structure.


Further reading
***************

.. todo::
    For more finite sized systems check out ...