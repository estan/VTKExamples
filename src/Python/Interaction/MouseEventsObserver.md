### Description
This example directly changes the observers of the vtkInteractor, which is an easy way to disable events or to add some simple callback functions. For a more general framework using vtkInteractorStyle see []([VTK/Examples/Python/Interaction/MouseEvents]). This specific example just disables the left mouse button of the vtkInteractorStyleTrackballCamera and prints a simple message instead.