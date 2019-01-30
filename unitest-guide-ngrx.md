# Unitest Guide - Angular Ngrx
## Action
Steps:
* Create an test action
* Check type and payload of action equal to the action you created

e.g.
```javascript
it('should create SEND_NOTIFICATION_FAILURE action', () => {
    const payload = new Response(undefined, { status: 404 })
    const action = new fromActions.SendNotificationFailure(payload);

    expect({ ...action }).toEqual({
      type: fromActions.SEND_NOTIFICATION_FAILURE,
      payload: payload
    });
});
```
## Reducer
Steps:
* Create the test action
* Declare the current state by initial state
* Declare the expect state with by reducer and current
* Check the data through the reducer is as expectation

e.g.
```javascript
it('LOAD_USER_FAILURE', () => {
    const action = new fromActions.LoadFailureAction(new Error('error'));
    const curState = { ...initialState, loading: true };
    const state = fromReducer.reducer(curState, action);
    expect(state).toEqual({
        ...initialState,
        error: 'error',
        loading: false
    });
});
```
ref: https://brianflove.com/2018/05/28/ngrx-testing-actions/

## Effect
Steps:
   * Fake the service and return function
   * New the effect with fake services(check construction)
   * Check the logic as expectation
 
The test cases:
* Check the effects return a right action that you expected
* Check the effects return a wrong action that you expected
* The metadata of an effect (dispatch attribute: true or not)

e.g.
```javascript
it('SendNotificationRequest success', () => {
    const completion = new fromActions.SendNotificationSuccess('Send Notification title Success');
    const action = new fromActions.SendNotificationRequest(
        {
          notification: { title: 'title', content: 'content', type: 'Info' },
          permissions: { id: [], tags: [] }
        });
    const actions = new Actions(hot('--a-', { a: action }));
    const effects = new NotifyDialogEffect(actions, helper.createAdminApieStub(null), null);
    const expected = cold('--b', { b: completion });
    expect(effects.NotifyRequestEffect$).toBeObservable(expected);
});

// fake api service for testing
export function createAdminApieStub(response: any) {
    const service = jasmine.createSpyObj('service', [
        'getUsersOnline',
        'reqNotification',
    ]);
    const isError = response instanceof Error || response instanceof Response;
    const serviceResponse = isError ? Observable.throw(response) : of(response);

    service.getUsersOnline.and.returnValue(serviceResponse);
    service.reqNotification.and.returnValue(serviceResponse);

    return service;
}


```
ref:
1. For hot, cold observable
 https://github.com/ReactiveX/rxjs/blob/master/doc/writing-marble-tests.md
2. For testing metadata https://github.com/ngrx/platform/blob/master/docs/effects/testing.md 

## Selector
Steps:
* Fake the input state
* Use selector.porjector to set the fake state to store
* Call the selector and check output as expectation

e.g.
```javascript
it('getPreviewData serach input test1 output userState1', () => {
    let result = (fromSelectors.getPreviewData.projector(userList));
    expect(result('test1')).toEqual([userState1]);
});
it('getPreviewData serach input TEST1 output userState1', () => {
    let result = (fromSelectors.getPreviewData.projector(userList));
    expect(result('TEST1')).toEqual([userState1]);
});
it('getPreviewData serach input TEST123 output null', () => {
    let result = (fromSelectors.getPreviewData.projector(userList));
    expect(result('TEST123')).toBeNull;
});
```
ref: Testing Approach #3: Projector
https://blog.angularindepth.com/how-i-test-my-ngrx-selectors-c50b1dc556bc

## Component
Steps:
* Use the real store, but fake the service and return data
* How to trigger the selector in component => 
dispatch the load success action with fake data.
* Check the component create success and method has been called
* Check the function work as expectation
* Integration for checking HTML preview

e.g.
```javascript
it('check functions have been called when component ngOnInit', () => {
    const action = new fromActions.LoadRequestAction();
    expect(store.dispatch).toHaveBeenCalledWith(action);
    expect(ss.enable).toHaveBeenCalled();
});

it('check if select null, the selected to be []', () => {
    component.onSelect({ selected: null });
    expect(component.selected).toEqual([]);
});

it('check the preview as expectation', () => {
    let fakeEvent = { target: { value: helper.userState1.id } };
    component.updateFilter(fakeEvent);
    fixture.detectChanges();
    const de = fixture.debugElement.query(By.css('.datatable-body-cell:nth-child(2)'));
    const el: HTMLElement = de.nativeElement;
    expect(el.innerHTML).toContain(helper.userState1.id);
    });
```
### others:
#### routerlink:
https://stackoverflow.com/questions/39623722/angular-2-final-release-router-unit-test

#### Form valid unitest
Notice: 
1. Validate by each field(correct & wrong)
2. Vaildate whole form

https://codecraft.tv/courses/angular/unit-testing/model-driven-forms/
