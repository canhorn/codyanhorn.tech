---
layout: post
author: Cody Merritt Anhorn
title: Design Patterns - Facade Pattern
categories: [blog, design_pattern]
tags: [Design Pattern, Facade]
---

The Facade Pattern is helpful for encapsulating complex architecture into a simpler and unified interface. This means that you can take something like file system access and make a unified abstraction that an application as a whole can use.

## What can a Facade do

To quote Wikipedia, A facade can:

> Improve the readability and usability of a software library by masking interaction with more complex components behind a single (and often simplified) API

> Provide a context-specific interface to more generic functionality (complete with context-specific input validation)

> Serve as a launching point for a broader refactor of monolithic or tightly-coupled systems in favor of more loosely-coupled code

## Usage

My personal usage of the Facade pattern is using it as an abstraction above the File System, as state in the introduction of this article. Using the facade pattern I am able to abstract the complexity of accessing a file system, but also able to create a simplified API that can be used to access files or directories. This also gets me the add benefit of having a single way to access the file system and being able to create an file system abstraction to other types of file/directory storage. 

By creating a virtual file system using the facade pattern, I am able to create a file system that is saving the file to a remote location, say an Azure Storage, or having everything in a single file for easy backup, or a document database. With this the application just needs to ask for the facade and use the provided API and the underlying storage can be switched out per the environment it might be deployed in.

## Example

Below is a simplified File System facade in C#.

***FileResolver.cs***
~~~ csharp
public interface FileResolver
{
    /// <summary>
    /// This will take in a file FullName,
    /// Try and create file,
    /// No text will be added.
    /// </summary>
    /// <param name="fileFullName"></param>
    /// <returns></returns>
    bool CreateFile(
        string fileFullName
    );
    IFileInfo GetFileInfo(
        string fileFullName
    );
    /// <summary>
    /// This will take in a file FullName, 
    /// Create if does not exist,
    /// Then append text to file.
    /// </summary>
    /// <param name="fileFullName"></param>
    /// <param name="text"></param>
    /// <returns></returns>
    bool AppendTextToFile(
        string fileFullName,
        string text
    );
    void DeleteFile(
        string fileFullName
    );
    bool DoesFileExist(
        string fileFullName
    );
    string GetFileText(
        string fileFullName
    );
    void WriteAllText(
        string fileFullName,
        string text
    );
}
~~~

***LocalFileSystemFileResolver.cs***
~~~ csharp
using EventHorizon.Zone.Core.Model.FileService;
using System.IO;

public class LocalFileSystemFileResolver : FileResolver
{
    public bool CreateFile(
        string fileFullName
    )
    {
        var fileInfo = new FileInfo(
            fileFullName
        );
        if (!fileInfo.Directory.Exists)
        {
            fileInfo.Directory.Create();
        }
        using (fileInfo.Create())
        {
            return fileInfo.Exists;
        }
    }

    public IFileInfo GetFileInfo(
        string fileFullName
    )
    {
        var fileInfo = new FileInfo(
            fileFullName
        );
        return new StandardFileInfo(
            fileInfo.Name,
            fileInfo.DirectoryName,
            fileInfo.FullName,
            fileInfo.Extension
        );
    }

    public bool AppendTextToFile(
        string fileFullName,
        string text
    )
    {
        var fileInfo = new FileInfo(
            fileFullName
        );
        if (DoesFileExist(
            fileFullName
        ))
        {
            using (var writer = fileInfo.AppendText())
            {
                writer.Write(
                    text
                );
            }
        }
        else
        {
            if (!fileInfo.Directory.Exists)
            {
                fileInfo.Directory.Create();
            }
            this.WriteAllText(
                fileFullName,
                text
            );
        }
        return fileInfo.Exists;
    }

    public void DeleteFile(
        string fileFullName
    ) => File.Delete(
        fileFullName
    );

    public bool DoesFileExist(
        string fileFullName
    ) => File.Exists(
        fileFullName
    );

    public string GetFileText(
        string fileFullName
    ) => File.ReadAllText(
        fileFullName
    );

    public void WriteAllText(
        string fileFullName,
        string text
    ) => File.WriteAllText(
        fileFullName,
        text
    );
}
~~~

## References

[Wikipedia - Facade pattern]([https://link](https://en.wikipedia.org/wiki/Facade_pattern))
