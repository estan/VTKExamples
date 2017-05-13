[VTKExamples](/home/)/[Cxx](/Cxx)/Images/ImagePermute

<img align="right" src="https://github.com/lorensen/VTKExamples/blob/gh-pages/Testing/Baseline/Images/TestImagePermute.png?raw=true" width="256" />

**ImagePermute.cxx**
```c++
#include <vtkSmartPointer.h>
#include <vtkImageData.h>
#include <vtkImageMapper3D.h>
#include <vtkImagePermute.h>
#include <vtkRenderWindow.h>
#include <vtkRenderWindowInteractor.h>
#include <vtkInteractorStyleImage.h>
#include <vtkRenderer.h>
#include <vtkImageActor.h>
#include <vtkImageMapper3D.h>
#include <vtkImageEllipsoidSource.h>
#include <vtkImageCast.h>

int main(int, char *[])
{
  // Create an image
  vtkSmartPointer<vtkImageEllipsoidSource > source = 
    vtkSmartPointer<vtkImageEllipsoidSource >::New();
  source->SetWholeExtent(0, 20, 0, 20, 0, 0);
  source->SetCenter(10,10,0);
  source->SetRadius(2,5,0);
  source->Update();

  vtkSmartPointer<vtkImagePermute> permuteFilter = 
    vtkSmartPointer<vtkImagePermute>::New();
  permuteFilter->SetInputConnection(source->GetOutputPort());
  permuteFilter->SetFilteredAxes(1,0,2);
  permuteFilter->Update();

  // Create actors
  vtkSmartPointer<vtkImageActor> originalActor =
    vtkSmartPointer<vtkImageActor>::New();
  originalActor->GetMapper()->SetInputConnection(
    source->GetOutputPort());

  vtkSmartPointer<vtkImageActor> permutedActor =
    vtkSmartPointer<vtkImageActor>::New();
  permutedActor->GetMapper()->SetInputConnection(
    permuteFilter->GetOutputPort());
  
  // Define viewport ranges
  // (xmin, ymin, xmax, ymax)
  double originalViewport[4] = {0.0, 0.0, 0.5, 1.0};
  double permutedViewport[4] = {0.5, 0.0, 1.0, 1.0};

  // Setup renderers
  vtkSmartPointer<vtkRenderer> originalRenderer =
    vtkSmartPointer<vtkRenderer>::New();
  originalRenderer->SetViewport(originalViewport);
  originalRenderer->AddActor(originalActor);
  originalRenderer->ResetCamera();
  originalRenderer->SetBackground(.4, .5, .6);

  vtkSmartPointer<vtkRenderer> permutedRenderer =
    vtkSmartPointer<vtkRenderer>::New();
  permutedRenderer->SetViewport(permutedViewport);
  permutedRenderer->AddActor(permutedActor);
  permutedRenderer->ResetCamera();
  permutedRenderer->SetBackground(.4, .5, .7);

  vtkSmartPointer<vtkRenderWindow> renderWindow =
    vtkSmartPointer<vtkRenderWindow>::New();
  renderWindow->SetSize(600, 300);
  renderWindow->AddRenderer(originalRenderer);
  renderWindow->AddRenderer(permutedRenderer);

  vtkSmartPointer<vtkRenderWindowInteractor> renderWindowInteractor =
    vtkSmartPointer<vtkRenderWindowInteractor>::New();
  vtkSmartPointer<vtkInteractorStyleImage> style =
    vtkSmartPointer<vtkInteractorStyleImage>::New();

  renderWindowInteractor->SetInteractorStyle(style);

  renderWindowInteractor->SetRenderWindow(renderWindow);
  renderWindowInteractor->Initialize();

  renderWindowInteractor->Start();
  
  return EXIT_SUCCESS;
}
```
**CMakeLists.txt**
```cmake
cmake_minimum_required(VERSION 2.8)
 
PROJECT(ImagePermute)
 
find_package(VTK REQUIRED)
include(${VTK_USE_FILE})
 
add_executable(ImagePermute MACOSX_BUNDLE ImagePermute.cxx)
 
target_link_libraries(ImagePermute ${VTK_LIBRARIES})
```

**Download and Build ImagePermute**

Click [here to download ImagePermute](https://github.com/lorensen/VTKWikiExamplesTarballs/raw/master/ImagePermute.tar) and its *CMakeLists.txt* file.
Once the *tarball ImagePermute.tar* has been downloaded and extracted,
```
cd ImagePermute/build 
```
If VTK is installed:
```
cmake ..
```
If VTK is not installed but compiled on your system, you will need to specify the path to your VTK build:
```
cmake -DVTK_DIR:PATH=/home/me/vtk_build ..
```
Build the project:
```
make
```
and run it:
```
./ImagePermute
```
**WINDOWS USERS PLEASE NOTE:** Be sure to add the VTK bin directory to your path. This will resolve the VTK dll's at run time.
