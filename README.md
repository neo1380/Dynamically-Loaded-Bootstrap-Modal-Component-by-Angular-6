# Dynamically-Loaded-Bootstrap-Modal-Component-by-Angular-6
Dynamically Loaded Bootstrap Modal Component by Angular 6
#dynamic modal dialog # dynamic modal #display component dynamically #Modal directive #angular 6 modal component #angular 6 modal dialog

Create the application using the Angular CLI command.

    ng new modal
Update the index.html file to reference the Bootstrap Css - using the CDN.

    <link rel="stylesheet" href="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.7/css/bootstrap.min.css">
    <script src="https://ajax.googleapis.com/ajax/libs/jquery/3.3.1/jquery.min.js"></script>
    <script src="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.7/js/bootstrap.min.js"></script>

App Component
Keep two buttons in app.componenet.html for loading two different modal popups.

app.component.html

    <div class="col-md-2">
        <button type="button" 
          class="btn btn-success  btn-block"
          modal-box 
          [title]='"Demo Modal 1"'
          [componentData]="data1"
          [componentName]="'Demo1Component'">
          Demo Modal 1
        </button>
      </div>  

      <div class="col-md-2">
        <button type="button" 
          class="btn btn-danger  btn-block"
          modal-box 
          [title]='"Demo Modal 2"'
          [componentData]="data2"
          [componentName]="'Demo2Component'">
          Demo Modal 2
        </button>
      </div>
    app.component.ts

    export class AppComponent {
      data1 : string;
      data2 : string;


      ngOnInit() {
        this.data1="The content is displayed from Demo1 component";
        this.data2="The content is displayed from Demo2 component";
      }

    }
app.module.ts

    import { BrowserModule } from '@angular/platform-browser';
    import { NgModule } from '@angular/core';
    import { HttpClientModule } from '@angular/common/http';

    import { AppComponent } from './app.component';
    import { Demo1Component } from './demo1/demo1.component';
    import { Demo2Component } from './demo2/demo2.component';
    import { ModalDialogComponent } from './modal-dialog/modal-dialog.component';
    import { ModalDirective } from './modal.directive';
    import { DatacontainerDirective } from './modal-dialog/datacontainer.directive';
    import { ComponentLoaderService } from './component-loader.service';

    @NgModule({
      declarations: [
        AppComponent,
        Demo1Component,
        Demo2Component,
        ModalDialogComponent,
        ModalDirective,
        DatacontainerDirective
      ],  
      providers: [ComponentLoaderService],
      imports: [
        BrowserModule,
        HttpClientModule
      ],
      entryComponents: [
        Demo1Component,
        Demo2Component,
        ModalDialogComponent
      ],
      bootstrap: [AppComponent]
    })
    export class AppModule { }

entryComponent is used for components declaration which will be loaded dynamically.

Modal Directive
modal.directive.ts

    import { Directive, Input, Output, EventEmitter, ElementRef, HostListener, Renderer2, ViewContainerRef, ComponentFactoryResolver } from '@angular/core';
    import { ModalDialogComponent } from './modal-dialog/modal-dialog.component';

    @Directive({
      selector: '[modal-box]'
    })
    export class ModalDirective {
      @Input() title: string;
      @Input() componentData: string;
      @Input() componentName: string;


      @HostListener('click', ['$event'])

      /* modal create */
      openModal() {
        this.createModalDialog(ModalDialogComponent);    
      }

      constructor(
            private el: ElementRef,
            private ren: Renderer2,
            private viewContainer: ViewContainerRef,
            private componentFactoryResolver: ComponentFactoryResolver
      ) { }

      ngOnInit() {
      }

      createModalDialog(modalDialogComponent) {
        this.viewContainer.clear();
        const modalDialogComponentFactory = this.componentFactoryResolver.resolveComponentFactory(modalDialogComponent);
        const modalDialogComponentRef = this.viewContainer.createComponent(modalDialogComponentFactory);
        modalDialogComponentRef.instance['title'] = this.title;
        modalDialogComponentRef.instance['componentData'] = this.componentData;
        modalDialogComponentRef.instance['componentName'] = this.componentName;

        return modalDialogComponentRef;
      }


    }
createModalDialog() function is used to create modal dialog componenet dynamically. You have to send modalDialogComponent through resolveComponenetFactory to create factory then will create component using createComponent function. We are fetching values of title, componentData, componentName from buttons which were declared in app.componenet.html. These value sends using modalDialogComponentRef.instance to modal-dialog.componenet.ts. 

Modal Dialog Component
modal-dialog.component.css

    .modal{
        display: block;
    }
modal-dialog.component.html

    <!-- Modal -->
    <div class="modal fade in" role="dialog" tabindex="-1" aria-hidden="true" #thisModal>
      <div class="modal-dialog">

        <!-- Modal content-->
        <div class="modal-content">
          <div class="modal-header">
            <button type="button" class="close" (click)="closeModal()"><span aria-hidden="true">&times;</span><span class="sr-only">Close</span></button>
            <h4 class="modal-title">{{ title }}</h4>
          </div>
          <div class="modal-body">
            <div #datacontainer></div>
          </div>
          <div class="modal-footer">
            <button type="button" class="btn btn-default" (click)="closeModal()">Close</button>
          </div>
        </div>

      </div>

modal-dialog.component.ts

    import {
      Component, Input, OnInit,
      ComponentFactoryResolver, ViewContainerRef, ViewChild, ElementRef, Renderer2, AfterContentInit
    } from '@angular/core';
    import { ComponentLoaderService } from '../component-loader.service';


    @Component({
      selector: 'modal-dialog',
      templateUrl: './modal-dialog.component.html',
      styleUrls: ['./modal-dialog.component.css']
    })

    export class ModalDialogComponent implements OnInit, AfterContentInit {
      @Input() title: string;
      @Input() componentData: string;
      @Input() componentName: any;
      public name : any;

      @ViewChild('datacontainer', { read: ViewContainerRef }) entry: ViewContainerRef;

      constructor(
        private el: ElementRef,
        private ren: Renderer2,
        private viewContainer: ViewContainerRef,
        private resolver: ComponentFactoryResolver,
        private loaderService :ComponentLoaderService
      ) {}

      public div = this.ren.createElement('div'); 

      ngOnInit() {      

      }

      ngAfterContentInit(){
        this.ren.addClass(this.el.nativeElement.ownerDocument.body, 'modal-open');
        this.ren.appendChild(this.el.nativeElement, this.div);
        this.ren.setAttribute(this.div , 'class', 'modal-backdrop fade in');
        this.createModalPopup();
      }

      createModalPopup(){
        const name = this.loaderService.getComponent(this.componentName);
        console.log("Component Name => ",name);
        const myFactory = this.resolver.resolveComponentFactory(<any>name);
        const myRef = this.entry.createComponent(myFactory);
        myRef.instance['data'] = this.componentData;
      }

      closeModal() {
        this.ren.removeClass(this.el.nativeElement.ownerDocument.body, 'modal-open');
        this.ren.removeChild(this.el.nativeElement, this.div);
        this.el.nativeElement.remove();
      }


    }

this.ren.addClass is used to add 'modal-open' class on creating modal-dialog. this.ren.appendChild is used append div in body. this.ren.setAttribute is used to add class name 'modal-backdrop fade in' in div which shows when modal-dialog opens for overlay. 

this.createModalPopup() is called for loading component dynamically. Component is coming from buttons of app.componenet.html page and this.componentName is used to hold componenent name. loaderService is used to pass a component according to the component name. That component is passing to the resolveComponentFactory, createComponent function to create the component accordingly. We are passing the componentData using myRef.instance['data'] to component like demo1 or demo2.

That data is fetching from demo1 and demo2 component using @Input() and displaing it from html file.

closeModal() is used to remove the modal. this.el.nativeElement.remove() is used to remove DOM element.
