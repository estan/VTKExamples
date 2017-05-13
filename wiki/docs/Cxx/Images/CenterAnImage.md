[VTKExamples](/home/)/[Cxx](/Cxx)/Images/CenterAnImage

<img align="right" src="https://github.com/lorensen/VTKExamples/blob/gh-pages/Testing/Baseline/Images/TestCenterAnImage.png?raw=true" width="256" />

### Description
This moves (0,0) from the bottom left corner of the image to the center of the image.

**CenterAnImage.cxx**
```c++
#include <vtkVersion.h>
#include <vtkSmartPointer.h>

#include <vtkActor.h>
#include <vtkBorderRepresentation.h>
#include <vtkBorderWidget.h>
#include <vtkCommand.h>
#include <vtkImageActor.h>
#include <vtkImageMapper3D.h>
#include <vtkImageChangeInformation.h>
#include <vtkImageClip.h>
#include <vtkImageData.h>
#include <vtkInteractorStyleImage.h>
#include <vtkJPEGReader.h>
#include <vtkMath.h>
#include <vtkProperty2D.h>
#include <vtkRenderWindow.h>
#include <vtkRenderWindowInteractor.h>
#include <vtkRenderer.h>
#include <vtkXMLPolyDataReader.h>

class vtkBorderCallback : public vtkCommand
{
public:
  vtkBorderCallback(){}

  static vtkBorderCallback *New()
  {
    return new vtkBorderCallback;
  }

  virtual void Execute(vtkObject *caller, unsigned long, void*);

  void SetRenderer(vtkSmartPointer<vtkRenderer> renderer)
  {
    this->Renderer = renderer;
  }
  void SetImageActor(vtkSmartPointer<vtkImageActor> actor)
  {
    this->ImageActor = actor;
  }

private:
  vtkSmartPointer<vtkRenderer>   Renderer;
  vtkSmartPointer<vtkImageActor> ImageActor;
};

int main(int argc, char* argv[])
{
  if ( argc != 2 )
  {
    std::cerr << "Usage: " << argv[0]
              << " Required parameters: Filename" << std::endl;
    return EXIT_FAILURE;
  }

  std::string inputFilename = argv[1];

  // Read the image
  vtkSmartPointer<vtkJPEGReader> jPEGReader =
    vtkSmartPointer<vtkJPEGReader>::New();

  if( !jPEGReader->CanReadFile( inputFilename.c_str() ) )
  {
    cout << "Error: cannot read " << inputFilename.c_str() << std::endl;
    return EXIT_FAILURE;
  }

  jPEGReader->SetFileName ( inputFilename.c_str() );
  jPEGReader->Update();

  // Shift the image center to (0,0)
  int dims[3];
  jPEGReader->GetOutput()->GetDimensions(dims);

  vtkSmartPointer<vtkImageChangeInformation> changeInformation =
    vtkSmartPointer<vtkImageChangeInformation>::New();
  changeInformation->SetInputConnection(jPEGReader->GetOutputPort());
  changeInformation->CenterImageOn();
  changeInformation->Update();

  vtkSmartPointer<vtkImageData> image =
    changeInformation->GetOutput();

  vtkSmartPointer<vtkImageActor> imageActor =
    vtkSmartPointer<vtkImageActor>::New();
#if VTK_MAJOR_VERSION <= 5
  imageActor->SetInput(image);
#else
  imageActor->GetMapper()->SetInputData(image);
#endif

  vtkSmartPointer<vtkRenderWindow> renderWindow =
    vtkSmartPointer<vtkRenderWindow>::New();

  vtkSmartPointer<vtkRenderWindowInteractor> interactor =
    vtkSmartPointer<vtkRenderWindowInteractor>::New();

  vtkSmartPointer<vtkInteractorStyleImage> style =
    vtkSmartPointer<vtkInteractorStyleImage>::New();
  interactor->SetInteractorStyle( style );

  vtkSmartPointer<vtkBorderWidget> borderWidget =
    vtkSmartPointer<vtkBorderWidget>::New();
  borderWidget->SetInteractor(interactor);
  static_cast<vtkBorderRepresentation*>(borderWidget->GetRepresentation())->GetBorderProperty()->SetColor(0,1,0);
  borderWidget->SelectableOff();

  interactor->SetRenderWindow(renderWindow);

  // Setup both renderers
  vtkSmartPointer<vtkRenderer> renderer =
    vtkSmartPointer<vtkRenderer>::New();
  renderer->SetBackground(1,0,0);
  renderWindow->AddRenderer(renderer);

  renderer->AddActor(imageActor);

  renderer->ResetCamera();

  vtkSmartPointer<vtkBorderCallback> borderCallback =
    vtkSmartPointer<vtkBorderCallback>::New();
  borderCallback->SetRenderer(renderer);
  borderCallback->SetImageActor(imageActor);

  borderWidget->AddObserver(vtkCommand::InteractionEvent,borderCallback);
  borderWidget->On();
  renderWindow->Render();
  interactor->Start();

  return EXIT_SUCCESS;
}

void vtkBorderCallback::Execute(vtkObject *caller, unsigned long, void*)
{

  vtkBorderWidget *borderWidget =
    reinterpret_cast<vtkBorderWidget*>(caller);

  // Get the world coordinates of the two corners of the box
  vtkCoordinate* lowerLeftCoordinate =
    static_cast<vtkBorderRepresentation*>(
      borderWidget->GetRepresentation())->GetPositionCoordinate();
  double* lowerLeft =
    lowerLeftCoordinate->GetComputedWorldValue(this->Renderer);
  std::cout << "Lower left coordinate: "
            << lowerLeft[0] << ", "
            << lowerLeft[1] << ", "
            << lowerLeft[2] << std::endl;

  vtkCoordinate* upperRightCoordinate =
    static_cast<vtkBorderRepresentation*>(
      borderWidget->GetRepresentation())->GetPosition2Coordinate();
  double* upperRight =
    upperRightCoordinate->GetComputedWorldValue(this->Renderer);
  std::cout << "Upper right coordinate: "
            << upperRight[0] << ", "
            << upperRight[1] << ", "
            << upperRight[2] << std::endl;

  double* bounds = this->ImageActor->GetBounds();
  double xmin = bounds[0];
  double xmax = bounds[1];
  double ymin = bounds[2];
  double ymax = bounds[3];

  if( (lowerLeft[0] > xmin) &&
      (upperRight[0] < xmax) &&
      (lowerLeft[1] > ymin) &&
      (upperRight[1] < ymax) )
  {
    //std::cout << "box is inside image" << std::endl;
    //std::cout << "xmin: " << xmin << " xmax: " << xmax << " ymin: " << ymin << " ymax: " << ymax << std::endl;
  }
  else
  {
    std::cout << "box is NOT inside image" << std::endl;
  }
}
```
**CMakeLists.txt**
```cmake
cmake_minimum_required(VERSION 2.8)
 
PROJECT(CenterAnImage)
 
find_package(VTK REQUIRED)
include(${VTK_USE_FILE})
 
add_executable(CenterAnImage MACOSX_BUNDLE CenterAnImage.cxx)
 
target_link_libraries(CenterAnImage ${VTK_LIBRARIES})
```

**Download and Build CenterAnImage**

Click [here to download CenterAnImage](https://github.com/lorensen/VTKWikiExamplesTarballs/raw/master/CenterAnImage.tar) and its *CMakeLists.txt* file.
Once the *tarball CenterAnImage.tar* has been downloaded and extracted,
```
cd CenterAnImage/build 
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
./CenterAnImage
```
**WINDOWS USERS PLEASE NOTE:** Be sure to add the VTK bin directory to your path. This will resolve the VTK dll's at run time.
