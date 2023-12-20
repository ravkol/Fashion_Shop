# Getting Started

Welcome to your new project.

It contains these folders and files, following our recommended project layout:

File or Folder | Purpose
---------|----------
`app/` | content for UI frontends goes here
`db/` | your domain models and data go here
`srv/` | your service models and code go here
`package.json` | project metadata and configuration
`readme.md` | this getting started guide


## Next Steps

- Open a new terminal and run `cds watch` 
- (in VS Code simply choose _**Terminal** > Run Task > cds watch_)
- Start adding content, for example, a [db/schema.cds](db/schema.cds).


## Learn More

Learn more at https://cap.cloud.sap/docs/get-started/.


###########Final fashionShop.cds################################
namespace app.fashionShop;
using { Currency } from '@sap/cds/common';
type Flag: String(1);

entity Sections {
    key id : UUID @(title: 'Section ID');
        name: String(16) @(title: 'Section Name');
        description: String(64)@(title: 'Section Description');   
                }
entity Fashion_Types {
    key id : UUID @(title: 'Fashion Type ID');
        section: Association to Sections @(title: 'Section ID');
        typename: String(16) @(title: 'Fashion Type Name');
        description: String(64) @(title: 'Fashion Type Description');   
                      }

entity Fashion_Items {
    key id : UUID @(title: 'Fashion Item ID');
        fashionType: Association to Fashion_Types @(title: 'Fashion Type ID');
        itemname: String(16) @(title: 'Fashion Item Name');
        brand: String(16) @(title: 'Brand');
        size:  String(8) @(title: 'Size');
        material: String(16) @(title: 'Material');
        price: String(10) @(title: 'Price');
        Currency: Currency @(title: 'Currency');
        isAvailable: Flag @(title: 'IsAvailable?');         
                    }

// View Creation for Fashion shop

view YC_FashionShop as select from Fashion_Items as fItem

{
   fItem.fashionType.section.id as sectionId,
   fItem.fashionType.section.name as sectionName,
   fItem.fashionType.section.description as sectionDesc,
   fItem.fashionType.id as fashionTypeId,
   fItem.fashionType.typename as fashionTypeName,
   fItem.fashionType.description as fashionTypeDesc,
   fItem.id as fashionItemId,
   fItem.itemname as fashionItemName,
   fItem.brand as brand,
   fItem.size as size,
   fItem.material as material,
   fItem.price as price,
   fItem.Currency as Currency,
   //fItem.isAvailable as isAvailable,
   concat( fItem.brand, concat ( ' ' ,fItem.itemname)) as itemDetails : String(32),
   case 
       when fItem.price >=500 then 'Premium'
       when fItem.price >=100 and fItem.price < 500 then 'Mid-Range'
       else 'Low-Range'
       end as PriceRange : String(15)

} where fItem.isAvailable = 'X';

view YC_FashionType_Valuehelp as select from Fashion_Types as fType
{
    fType.id as fashionTypeID,
    fType.typename as fashionTypeName,
    fType.section.name as sectionName
};
##########Final fashionShop_srv.cds###################
using app.fashionShop from '../db/fashionShop';

service FashionShop_Service {
    entity Sections        as projection on fashionShop.Sections;

    @cds.redirection.target: true
    entity Fashion_Types   as projection on fashionShop.Fashion_Types;

    entity fashion_Items   as projection on fashionShop.Fashion_Items;
    entity Srv_FashionShop as projection on fashionShop.YC_FashionShop;
    entity F4_FashionType  as projection on fashionShop.YC_FashionType_Valuehelp;
}

@odata.draft.enabled
annotate fashionShop.Fashion_Items with @(UI: {

    CreateHidden           : false,
    DeleteHidden           : false,
    UpdateHidden           : false,

    HeaderInfo             : {
        $Type         : 'UI.HeaderInfoType',
        TypeName      : 'Online Fashion Shop',
        TypeNamePlural: 'Online Fashion Shop',
        Title         : {Value: itemname},
        Description   : {Value: 'Online Fashion Shop'}
    },

    SelectionFields        : [

        fashionType_id,
        itemname,
        brand,
        size,
        price

    ],

    LineItem               : [
        {Value: fashionType.section.name},
        {Value: fashionType.typename},
        {Value: itemname},
        {Value: brand},
        {Value: size},
        {Value: price},
        {Value: Currency_code}

    ],

    Facets                 : [
        {
            $Type : 'UI.CollectionFacet',
            ID    : '1',
            Label : 'Fashion Type & Section',
            Facets: [{
                $Type : 'UI.ReferenceFacet',
                Target: '@UI.FieldGroup#TypeSection',
            }],
        },
        {
            $Type : 'UI.CollectionFacet',
            ID    : '2',
            Label : 'Fashion Item',
            Facets: [{
                $Type : 'UI.ReferenceFacet',
                Target: '@UI.FieldGroup#FItem',
            }],
        }
    ],

    FieldGroup #TypeSection: {Data: [

        {Value: fashionType_id},
        {Value: fashionType.typename,
        ![@Common.FieldControl]: #ReadOnly},
        {Value: fashionType.description,
        ![@Common.FieldControl]: #ReadOnly},
        {Value: fashionType.section.id,
        ![@Common.FieldControl]: #ReadOnly},
        {Value: fashionType.section.name,
        ![@Common.FieldControl]: #ReadOnly},
    ]

    },
    FieldGroup #FItem      : {Data: [

        {Value: id},
        {Value: itemname},
        {Value: brand},
        {Value: material},
        {Value: size},
        {Value: price},
        {Value: Currency_code},
        {Value: isAvailable}
    ]}
});

annotate FashionShop_Service.fashion_Items with {

    fashionType @(

        title         : 'Fashion Type',
        sap.value.list: 'Fashion- Values',
        Common        : {

            ValueListWithFixedValues,
            ValueList: {
                CollectionPath: 'F4_FashionType',
                Parameters    : [
                    {
                        $Type            : 'Common.ValueListParameterInOut',
                        ValueListProperty: 'fashionTypeID',
                        LocalDataProperty: fashionType_id
                    },
                    {
                        $Type            : 'Common.ValueListParameterDisplayOnly',
                        ValueListProperty: 'sectionName'

                    },
                    {
                        $Type            : 'Common.ValueListParameterDisplayOnly',
                        ValueListProperty: 'fashionTypeName'

                    }

                ]
            },


        }

    )

};
############### Final MTA.yaml######################
_schema-version: '3.1'
ID: Fashion_Shop
version: 1.0.0
description: "A simple CAP project."
parameters:
  enable-parallel-deployments: true
build-parameters:
  before-all:
    - builder: custom
      commands:
        - npx cds build --production
modules:
  - name: Fashion_Shop-srv
    type: nodejs
    path: gen/srv
    parameters:
      buildpack: nodejs_buildpack
    build-parameters:
      builder: npm
    provides:
      - name: srv-api # required by consumers of CAP services (e.g. approuter)
        properties:
          srv-url: ${default-url}
    requires:
      - name: Fashion_Shop-db

  - name: Fashion_Shop-db-deployer
    type: hdb
    path: db #gen/db
    parameters:
      buildpack: nodejs_buildpack
    requires:
      - name: Fashion_Shop-db

resources:
  - name: Fashion_Shop-db
    type: com.sap.xs.hdi-container
    parameters:
      service: hana
      service-plan: hdi-shared

############## Final Package.json######################
{
  "name": "Fashion_Shop",
  "version": "1.0.0",
  "description": "A simple CAP project.",
  "repository": "<Add your repository here>",
  "license": "UNLICENSED",
  "private": true,
  "dependencies": {
    "@sap/cds": "^7",
    "express": "^4",
    "@sap/cds-hana": "^2"
  },
  "scripts": {
    "start": "cds-serve"
  },
  "cds": {
    "build": {
      "tasks": [
        {
          "for": "hana",
          "dest": "../db"
        },
        {
          "for": "node-cf"
        }
      ]
    },
    "requires": {
      "db": {
        "kind": "hana-cloud"
      }
    }
  }
}
######################################## Fashion_Shop
