# PNRR-web

This project was generated with [Angular CLI](https://github.com/angular/angular-cli) version 13.0.3. The following libraries are used in combination with Angular:

- [PrimeNG][primeng]
- [Deloitte Digital Design System][digitaldesignsystem]

## Development server

Run `ng serve` for a dev server. Navigate to `http://localhost:4200/`. The app will automatically reload if you change any of the source files.

## Code scaffolding

Run `ng generate component component-name` to generate a new component. You can also use `ng generate directive|pipe|service|class|guard|interface|enum|module`.

## Build

Run `ng build` to build the project. The build artifacts will be stored in the `dist/` directory.

## Running unit tests

Run `ng test` to execute the unit tests via [Karma](https://karma-runner.github.io).

## Running end-to-end tests

Run `ng e2e` to execute the end-to-end tests via a platform of your choice. To use this command, you need to first add a package that implements end-to-end testing capabilities.

## Further help

To get more help on the Angular CLI use `ng help` or go check out the [Angular CLI Overview and Command Reference](https://angular.io/cli) page.

## installation

In order to run the application you need to have access to the DDS library. To gain access contact the project administrator. After you gain the access, you have to follow the tutorial of the [DDS library setup instructions](https://design.deloitte.com/for-developers/setup-instructions), the steps 01 and 05.

Delete the commented part (lines 3 to 13) in the .npmrc file.

When the setup of DDS Library is completed, you have to run `npm install` to install all the required dependencies and then `ng serve` to launch the application.

After running `npm install`, undo the changes made to the .npmrc file.

## Contribution

For contribution guidelines follow the [contributing guide][contributing]




[contributing]: CONTRIBUTING.md
[primeng]: https://www.primefaces.org/primeng/
[digitaldesignsystem]: https://design.deloitte.com/
