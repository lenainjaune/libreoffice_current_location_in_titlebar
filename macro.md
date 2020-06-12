```vba
REM  *****  BASIC  *****

' TODO : try to migrate this script to javascript language in the case where it can giving the same functionnalities without storing the decode URL in a temporarily file

' Important : this code MUST be placed on ~/.config/libreoffice/4/user/basic/Standard/ macro

' equivalent topics : 
' - https://ask.libreoffice.org/en/question/11292/how-to-display-full-file-path-in-window-title/
' - https://ask.libreoffice.org/en/question/44280/how-to-view-full-path-of-open-document/

Sub on_DocLoad(event As Object)
	' Important : This Sub MUST be connected to the "Open Document" event of LibreOffice.
	' https://ask.libreoffice.org/en/question/145481/struggling-to-auto-run-a-macro/
	' The URL in LibreOffice : https://help.libreoffice.org/3.6/Basic/Basic_Glossary/fr#Notation_URL
	' The events in LibreOffice : https://help.libreoffice.org/4.2/Basic/Event-Driven_Macros
	
	' A big problem :
	' A document "with space and accentué.odt" from "smb://nas.local/foo bar" network share
	' when converted with ConvertFromURL() gives : 
	' "smb://nas.local/test/foo%20bar/with%20space%20and%20accentu%C3%A9.odt" which is hard to read. 
	' When I try to decode it with https://www.url-encode-decode.com/ it gives :
	' "smb://nas.local/test/to to/with space and accentué.odt" which is easy to read.
	' => solution : decode URL before display it
	
	' TODO : find a native way to decode URL without storing the decoded URL in a temporarily file or implement a function to replace %xx (https://www.w3schools.com/tags/ref_urlencode.ASP)
	
	' New document => nothing to do
	If ThisComponent.Location = "" Then 
		Exit Sub
	End If

	Dim locationInTitleBar As String
	Dim tempFileLocation As String
	Dim currentController As Object
	Dim frame As Object
	
	tempFileLocation = Environ("HOME") & "/URL_converted"

	' URL Decode with bash to temporarily file
	' Based on : https://stackoverflow.com/questions/6250698/how-to-decode-url-encoded-string-in-shell#37840948
	' Nota : I had just implemented the replace of % BUT not the replace of + (for space characters)
	cmd = 	"bash -c " & _
			Chr(34) & _
				"url=" & ThisComponent.Location & _
				" ; echo -e " & "${url//%/\\x}" & _
				" > " & tempFileLocation & _
			Chr(34) & _
		""			
	Shell(cmd, 0, "", true)
	
	' Get decoded URL from temporarily file
	FileNo = Freefile
	Open tempFileLocation For Input As #fileNo
	While Not Eof(#fileNo)
		Line Input #fileNo, locationInTitleBar
	WEnd
	Close #fileNo
	Kill(tempFileLocation)

	' Current location in title bar
	' https://ask.libreoffice.org/en/question/147789/can-i-change-the-content-of-the-title-bar-from-a-macro/
	currentController = ThisComponent.getCurrentController()
	frame = currentController.getFrame()
	frame.Title = ConvertFromURL(locationInTitleBar) & " - " & getDocumentType(thisComponent)
End Sub

' Based on https://wiki.openoffice.org/wiki/Currently_active_document
Function getDocumentType(component As Object) As String
	If HasUnoInterfaces(component, "com.sun.star.lang.XServiceInfo") Then
		If thisComponent.supportsService ("com.sun.star.text.GenericTextDocument") Then
			getDocumentType = "LibreOffice Writer"
		ElseIf thisComponent.supportsService("com.sun.star.sheet.SpreadsheetDocument") Then
			getDocumentType = "LibreOffice Calc"
		ElseIf thisComponent.supportsService("com.sun.star.presentation.PresentationDocument") Then
			getDocumentType = "LibreOffice Impress"
		ElseIf thisComponent.supportsService("com.sun.star.drawing.GenericDrawingDocument") Then
			getDocumentType = "LibreOffice Draw"
		Else
			getDocumentType = "Unknown Document type"
		End If
	Else
		getDocumentType = "Not a document"
	End If
End Function
```
