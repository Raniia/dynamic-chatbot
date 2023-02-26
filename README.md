<h1>Dynamic Chatbot</h1>
<p>The dynamic chatbot helps solve the user's problem by giving him some dynamic questions and a list of answers. According to the user's answer, the dynamic chatbot will load the next question dynamically. </p>
<p>It is responsible for creating two dynamic components <strong>System component</strong> (which represents the question asked to the user) and the <strong> User Message Component </strong> (which represents the answer the user can choose from.
The System Component could be a question, a statement, a tutorial or even an image.
The User Message could be buttons where the user chooses a certain answer, a timer, an image selector, or a form in which the user should submit.

All these components (system & user) will be loaded dynamically according to the response returned from the backend.
</p>


<h2>Service - Map and Prepare components</h2>

```typescript
   /**
    * mapAndDrawComponents takes response from BE and answer from the user; Based on both of them, the dynamic chatbot creates 2 components, System message and User 
    * Component
    */
    public mapAndDrawComponents (response , answer?: any) {

        let components: any = [];

        components = this.createComponents(response , answer);

        if (components) {

            this._components = this._components.concat(components);

            this.componentsUpdated.emit(cloneDeep(components));
        }
    }
    
```

<h2>Decision Tree Component- Host Element including decision tree UI</h2>
    
```typescript

    /**
     * startListeningForSubjects 
     * The previous function (mapAndDrawComponents) emits the updated component tree.
     * In our decision tree component, we subscribe on the componentsUpdated property and pass the host element to the function that's responsible for drawing     * * 
     * component to inject the dynamic component into the host element 
     */
    private startListeningForSubjects () {
        this.componentsUpdatedSubscription = this.service
            .componentsUpdated
            .subscribe((component) => {
                this.components = component;
                this.componentLoaderService.drawComponent(this.components, this.componentHost);
                this.tooltipData();
            });
        this.removeComponentsSubscription = this.service
            .componentsRemove
            .asObservable()
            .subscribe((stepNumber: number) => {

                this.componentLoaderService.removeComponents(this.componentHost, stepNumber);
            });
    }
```
 <h2>Service - Draw and Remove component </h2>
        
```typescript

    /**
     * draw list of component system component and answers component in view 
     * creates components and injects them into the host element passed from the decision tree component
     * @param components
     * @param componentHost
     */
    public drawComponent (components: any, componentHost: ViewContainerDirective) {
        if (componentHost) {
            components.forEach(component => {
                const componentItem = component;
                const componentFactory = this.componentFactoryResolver.resolveComponentFactory(componentItem.component);
                const viewContainerRef = componentHost.viewContainerRef;
                const componentRef = viewContainerRef.createComponent(componentFactory);
                (<ComponentData>componentRef.instance).data = componentItem.data;
                if (typeof componentItem.step !== 'undefined') {
                    (<ComponentData>componentRef.instance).step = componentItem.step;
                }
            });
        }
    }
    
     /**
     * remove all the components if user selected old step and not the currentstep
     * all components after that step will removed
     * If the user wishes to change his answer to any question in the middle, the components after this question will be removed and start loading components 
     * dynamically according to the new answer
     * @param componentRef
     * @param stepNumber {number}
     */
    public removeComponents (componentRef, stepNumber: number) {
        const startIndex = findIndex(this._components, (component: ComponentItem) => {

            return component.step === stepNumber;
        }) + 1;
        const lastIndex = this._components.length - 1;
        this._components.splice(startIndex , this._components.length);
        if (this._step) {
            this._step = stepNumber ;
        }
        for (let i = lastIndex; i >= startIndex; i--) {
            this.removeComponent(componentRef, i);
        }
    }
```

<h2>Template Passed as host element in order to inject dynamic components </h2>

```html

 <ng-template view-container></ng-template>
 
```

