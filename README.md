# Lab-9-Polymorphic-Drawing

Lab 9 - Polymorphic Shapes

Note: we are giving you test cases and a sample main driver which means that your code will need to exactly match the names, etc. that we give you.

General Learning Objectives

Use a modern operating system and utilities.

Use an integrated development environment to develop a program.

Solve problems and develop programs using the control structures of sequence, selection, and repetition, following a disciplined approach.

Objectives
Practice setting up a solution with a class-library, unit tests, and a main driver.
Practice using polymorphism.
Practice using an abstract class.
Creating a basic textual graphics library.
Practice with properties in C#.
Use your graphics library to draw a picture with console graphics.
Step 1: Read and Understand This Overview

In this lab you will write a console graphics library. A drawing will be described in code as a collection of shape objects of various types, dimensions, colors, and patterns. Each object will satisfy an interface that requires it to know how to display itself (fill colors, etc. as well as knowing whether any given point is contained within the shape and knowing a rough bounding box for what points are worth asking about). (This interface will be given to you.)

The shape types differ in terms of the data required to represent the dimensions of the shape (e.g. a rectangle has a top/left corner and a height and a width while a circle has a center and a radius) and also in the way that they decide whether a point is contained within the shape and what the dimensions of the bounding box should be. (Recall that this difference in behavior across objects of a shared interface is called "polymorphism" and it allows us to write more code at a higher level of abstraction.)

Despite the differences, all shape types must have the following data:

a foreground color
a background color
a display character

Since we cannot include data in the shared interface, inheriting from a abstract class the holds the data and implements getters/setters for it will let us reduce duplication across the shape types. (This abstract class will be given to you.)

Now, consider the following main driver that uses the library (if you are unfamiliar with the initialization block following a constructor call, just imagine the code in the curly braces after each constructor call as if it were being run in the constructor, however, note that you can only initialize public members in this way).

Lab09.Main/SampleDrawing.cs
using Lab08;

List<IGraphic2D> shapes = new List<IGraphic2D> {
    new Circle(10, 10, 5) { BackgroundColor = ConsoleColor.DarkYellow, DisplayChar = ' ' },
    new Circle(8, 10, 1m) { BackgroundColor = ConsoleColor.White, ForegroundColor = ConsoleColor.Gray, DisplayChar = '.' },
    new Circle(12, 10, 1m) { BackgroundColor = ConsoleColor.White, ForegroundColor = ConsoleColor.Gray, DisplayChar = '.' },
    new Circle(8, 10, 0.5m) { BackgroundColor = ConsoleColor.Blue, ForegroundColor = ConsoleColor.DarkBlue, DisplayChar = '.' },
    new Circle(12, 10, 0.5m) { BackgroundColor = ConsoleColor.Blue, ForegroundColor = ConsoleColor.DarkBlue, DisplayChar = '.' },
    new Rectangle(8, 13, 4, 0.5m) { ForegroundColor = ConsoleColor.DarkGray, DisplayChar = 'v' },
    new Rectangle(8, 16, 4, 10) { ForegroundColor = ConsoleColor.DarkGreen, DisplayChar = '#' }
};

Console.Clear();
AbstractGraphic2D.Display(shapes);


The awesome thing is that actual drawing logic can be written in the interface file since it doesn't depend on knowing any of the details that distinguish shape types.

To understand how the library should work, read the following tests that will need to pass for the Rectangle and Circle classes that you will write:

Lab09.Tests/CircleTests.cs
using NUnit.Framework;

namespace Lab08.Tests;

public class CircleTests
{
    Circle circle;
    AbstractGraphic2D shape;

    [SetUp]
    public void Setup()
    {
        // should be x, y, and radius
        circle = new Circle(8, 10, 2);

        // should extend the abstract class
        shape = circle;
    }

    [Test]
    public void CircleHasCorrectDimensions()
    {
        Assert.AreEqual(8, circle.CenterX);
        Assert.AreEqual(10, circle.CenterY);
        Assert.AreEqual(2, circle.Radius);
    }

    [Test]
    public void HasCorrectBoundingBox()
    {
        Assert.AreEqual(8 - 2, shape.LowerBoundX);
        Assert.AreEqual(10 - 2, shape.LowerBoundY);
        Assert.AreEqual(8 + 2, shape.UpperBoundX);
        Assert.AreEqual(10 + 2, shape.UpperBoundY);
    }

    [Test]
    public void CenterIsIncluded()
    {
        Assert.IsTrue(shape.ContainsPoint(8, 10));
    }

    [Test]
    public void ContainsAllFourPointsOfTheCompass()
    {
        Assert.IsTrue(shape.ContainsPoint(8 - 2, 10));
        Assert.IsTrue(shape.ContainsPoint(8 + 2, 10));
        Assert.IsTrue(shape.ContainsPoint(8, 10 - 2));
        Assert.IsTrue(shape.ContainsPoint(8, 10 + 2));
    }

    [Test]
    public void ShouldNotContainFourCorners()
    {
        Assert.IsFalse(shape.ContainsPoint(8 - 2, 10 - 2));
        Assert.IsFalse(shape.ContainsPoint(8 + 2, 10 - 2));
        Assert.IsFalse(shape.ContainsPoint(8 - 2, 10 + 2));
        Assert.IsFalse(shape.ContainsPoint(8 + 2, 10 + 2));
    }
}

Lab09.Tests/RectangleTests.cs
using NUnit.Framework;

namespace Lab08.Tests;

public class RectangleTests
{
    Rectangle rectangle;
    AbstractGraphic2D shape;

    [SetUp]
    public void Setup()
    {
        rectangle = new Rectangle(3, 4, 5, 6);
        shape = rectangle;
    }

    [Test]
    public void EnsurePropertiesAreCorrect()
    {
        Assert.AreEqual(3, rectangle.Left);
        Assert.AreEqual(4, rectangle.Top);
        Assert.AreEqual(5, rectangle.Width);
        Assert.AreEqual(6, rectangle.Height);
    }

    [Test]
    public void CheckLowerBounds()
    {
        // lower bound is the smallest x that needs to be checked when drawing the shape
        Assert.AreEqual(3, shape.LowerBoundX);
        Assert.AreEqual(4, shape.LowerBoundY);
    }

    [Test]
    public void CheckUpperBounds()
    {
        // upper bound is the largest x that needs to be checked when drawing the shape
        Assert.AreEqual(3 + 5, shape.UpperBoundX);
        Assert.AreEqual(4 + 6, shape.UpperBoundY);
    }

    [Test]
    public void MiddleOfShapeIsIncluded()
    {
        Assert.IsTrue(shape.ContainsPoint(5.5m, 7));
    }

    [Test]
    public void CornersIncluded()
    {
        Assert.IsTrue(shape.ContainsPoint(3, 4));
        Assert.IsTrue(shape.ContainsPoint(8, 4));
        Assert.IsTrue(shape.ContainsPoint(3, 10));
        Assert.IsTrue(shape.ContainsPoint(8, 10));
    }

    [Test]
    public void OutsideOfCornersNotIncludedInShape()
    {
        Assert.IsFalse(shape.ContainsPoint(3 - 0.1m, 4));
        Assert.IsFalse(shape.ContainsPoint(8, 4 - 0.1m));
        Assert.IsFalse(shape.ContainsPoint(3, 10 + 0.1m));
        Assert.IsFalse(shape.ContainsPoint(8 + 0.1m, 10));
    }
}


You will use the provided interface and abstract class to implement Rectangle and Circle so as to pass the tests and make the sample drawing driver work correctly. You can optionally choose to create your own drawing with the library.

Step 2: Setup Basic Environment
Accept the GitHub Classroom Assignment.
Open a terminal.
Navigate to your code directory for the course (e.g.: cd my-cs1415-dir).
Run git clone {yourUrl} to make a local copy of your repository.
Run code {nameOfYourRepo} to open that directory in VS Code.
Open the integrated terminal in VS Code (Ctrl+`, or the Terminal menu -> New Terminal).
Step 3: Create the Library, Test, and Main Driver Projects

Note: Step 3 has already been completed in the starter code. These steps can be skipped if cloned the GitHub Classroom assignment. The steps are listed here for your reference, but you don't actually have to follow them.

Navigate to your repository folder in the VS Code terminal.
Create the following three projects:
dotnet new classlib -o Lab08
dotnet new nunit -o Lab08.Tests
dotnet new console -o Lab08.Main
Add references to the tests and main driver from the libary:
dotnet add Lab06.Tests reference Lab08
dotnet add Lab06.Main reference Lab08
Create a solution pointing to all three projects:
dotnet new sln
dotnet sln add Lab08 Lab08.Tests Lab08.Main
The sample test should run and pass:
dotnet test
Step 4: Read and Add the Starter Code to the Appropriate Files
Lab08/IGraphic2D.cs
namespace Lab08;

// Represents a graphical element (i.e. a shape) that can be printed to the
// console.
public interface IGraphic2D
{

    // draws the graphic on the screen; return true if successful
    bool Display();

}

Lab09/AbstractGraphic2D.cs
namespace Lab08;

// Implements the display char and color properties.
public abstract class AbstractGraphic2D : IGraphic2D
{
    // returns true if the given point is within (including the border)
    // of the shape
    public abstract bool ContainsPoint(decimal x, decimal y);

    // the following indicate a (possibly loose) bounding box for the element
    public abstract decimal LowerBoundX { get; }
    public abstract decimal UpperBoundX { get; }
    public abstract decimal LowerBoundY { get; }
    public abstract decimal UpperBoundY { get; }

    // the character to be displayed on cells within the image
    public char DisplayChar { get; set; }
    // the color of character to be displayed on cells within the image
    public ConsoleColor ForegroundColor { get; set; }
    // the background color of character to be displayed on cells within the image
    public ConsoleColor BackgroundColor { get; set; }

    // displays the given list of shapes
    public static void Display(List<IGraphic2D> shapes)
    {
        bool skippedSome = false;
        foreach (IGraphic2D shape in shapes)
        {
            if (!shape.Display())
            {
                skippedSome = true;
            }
        }
        if (skippedSome)
        {
            Console.WriteLine("warning: some cells skipped because  of too small buffer");
        }
    }

    public bool Display()
    {
        Console.ForegroundColor = ForegroundColor;
        Console.BackgroundColor = BackgroundColor;
        int lowX = (int)decimal.Floor(LowerBoundX);
        int lowY = (int)decimal.Floor(LowerBoundY);
        int highX = (int)decimal.Floor(UpperBoundX);
        int highY = (int)decimal.Floor(UpperBoundY);
        bool skippedSome = false;
        for (int row = lowY; row <= highY; row++)
        {
            for (int column = lowX; column <= highX; column++)
            {
                if (ContainsPoint(column, row))
                {
                    if (column < Console.BufferWidth && row < Console.BufferHeight)
                    {
                        Console.SetCursorPosition(column, row);
                        Console.Write(DisplayChar);
                    }
                    else
                    {
                        skippedSome = true;
                    }
                }
            }
        }
        Console.ForegroundColor = ConsoleColor.White;
        Console.BackgroundColor = ConsoleColor.Black;
        Console.SetCursorPosition(0, 0);
        return !skippedSome;
    }

}

Step 5: Create Stubs for Circle and Rectangle Classes

Put each in its own .cs file. Here is an example for Circle

Lab09/Circle.cs
namespace Lab08;

public class Circle : AbstractGraphic2D
{
    public override decimal LowerBoundX => -1;

    public override decimal UpperBoundX => -1;

    public override decimal LowerBoundY => -1;

    public override decimal UpperBoundY => -1;

    public override bool ContainsPoint(decimal x, decimal y)
    {
        return false;
    }
}

Step 6: Test Driven Development

Comment out all of the main driver and all tests and make sure that things can compile and run. Make sure all code is added to your repo and do a git commit.

Uncomment the fewest number of assertions possible to get a failing test case. (commit)

Write the minimum amount of code to get current tests passing. (commit)

‼️‼️‼️‼️‼️
NOTE Remember that (0,0) is the top left corner. So as the x value gets bigger, you are moving to the right; and as the y value gets bigger you are moving down. That means that UpperBoundY is not the highest point, but in fact the lowest point because it is the farthest from the top left corner.
‼️‼️‼️‼️‼️

If there are more commented-out tests, return to (2).

Once all tests are passing, uncomment the contents of the main driver and use dotnet run --project Lab08.Main to run the sample drawing. (Make sure that it looks as you would expect.)

(optional) Create your own drawing with the library. (Consider that you can use loops and math to create more interesting drawings.)

Make sure the final version of your code is all added, committed, and pushed to github.

Grading

Grading must be done in person or via Teams with a TA or instructor.

You may work in groups of two or three to complete this project, however, each individual will need to submit a repository.  Each individual should include comments in their code indicating which portions of the code they contributed to the final project. 

Getting a TA or instructor to sign-off on your lab before you leave is the easiest and best way to get the best grade possible.
