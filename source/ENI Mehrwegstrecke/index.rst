ENI Mehrwegstrecke
===================

**Description**

* The order is forwarded from SAP to team beverage, the assignment of customers => logistics center Team Beverage takes place through Team Beverage feedback from team beverage to our SAP system for actual quantities
* Reusable items are available in the SAP / PIN, but without conditions, a total of approximately 1,600 items

Price Interface
----------------

To show the prices of the Team Beverage (TB) articles (“Mehrweg“) in the shop, the price data have to be tranferred from the TB system to the shop via the iMan usinf the same file format like the LL24 price data files. The Team Beverage (TB) price data are in the same format like the price date files from LL-SAP.


Price Information
------------------

To show the prices of the Team Beverage (TB) articles (“Mehrweg“) in the shop, the price data have to be tranferred from the TB system to the shop via the iMan usinf the same file format like the LL24 price data files. The Team Beverage (TB) price data are in the same format like the price date files from LL-SAP.

::

    public function getSapListingMode()
    {
        if ($this->_sSapListingMode === null) {
            switch (Registry::getConfig()->getConfigParam('sBdsSapListing')) {
                case self::BDS_SAP_LISTING_MODE_LOGISTICCENTER:
                    $this->_sSapListingMode = self::BDS_SAP_LISTING_MODE_LOGISTICCENTER;
                    break;
                case self::BDS_SAP_LISTING_MODE_INDIVIDUAL:
                    $this->_sSapListingMode = self::BDS_SAP_LISTING_MODE_INDIVIDUAL;
                    break;
                default:
                    $this->_sSapListingMode = false;
                    break;
            }
        }
        return $this->_sSapListingMode;
    }

``getSapListingMode()`` is used in different modules and classes. To prevent complicated dependencies the idea is to bring the logic together.

Convert price logic to individual prices
------------------------------------------

Mehrwegstrecke Checkout
------------------------

**Description**

* In the shopping cart display a similar marking as for parcel - loading groups; new icon is needed
* Delivery calendar for this loading group out, but additional text note on the delivery of team beverage
* Display of prices from reusable price interface and not from our SAP

------------------------------------------------

.. function:: getPremiumSpiritsOrderType()
   	             
   :module: bds_split_order.Model.Basket.php

To check if the user has rights.

------------------------------------------------

.. function:: getFrozenArticleOrderType()
              
   :module: bds_split_order.Model.Basket.php
   
To check if the Basket is frozen.

------------------------------------------------

.. function:: isDeliveryCalendarDisabledForTKKBasket()
	      isDeliveryCalendarEnabled()
              
   :module: bds_lekkerland.Controller.BasketController.php
   
Checks if user calendar is disable for product.

------------------------------------------------

.. function:: getMehrwegStreckeOrderType()
              
   :module: bds_split_order.Model.Basket.php

Only when ``getPremiumSpiritsOrderType() == ture && getFrozenArticleOrderType() == false`` then go to ``getMehrwegStreckeOrderType()``.

-----------------------------------------------

::

 public function isMehrwegStreckeArtikel()
  {
      return $this->getArticleType() == Basket::BDS_BASKET_TYPE_MEHRWEGSTRECKE;
  }

Content Adjustments
----------------------

As a ENI shop user i want to see a content page for the Mehrwegstrecke to explain the concept the content page should be like the 'Premium Spirituosen' in LL24, constisting of an image and some text

``var/configuration/shops/2.yaml``::

 moduleSettings:
      aImexImportCategoryTemplates:
        group: BDS_IMEX
        type: aarr
        value:
          '2_00003_1': page/list/list_pspr_landing.tpl
          '2_00011_1': page/list/alt_list_content.tpl
      aImexImportCategoryMainCategories:
        group: BDS_IMEX
        type: aarr
        value:
          P0010: Getränke
          P0020: Lebensmittel

configuration of the alternative template for Mehrwegstrecke page

Delivered by Lekkerland
------------------------

**Description**

The customer can choose if the "Mehrwegstrecke" basket/order will be delivered by Team Beverage or the LL logistic center. A new parameter has to be send to the SAP system within the simulation / order request.

---------------------------------------------------------------------------

::

  private function getBstsz($sType, $blLekkerlandLogisticOrder = false)
    {
        $sBSTZD = '';
        if($sType == Basket::BDS_BASKET_TYPE_MEHRWEGSTRECKE && $blLekkerlandLogisticOrder) {
            $sBSTZD = Basket::BDS_MEHRWEGSTRECKE_BSTZD;
        }
        return $sBSTZD;
    }

The new field in the simulation and save order request is called BSTZD. If the customer wants to order his "Mehrwegstrecke" articles of the Lekkerland logistics the field has to be filled with the value „EL“ (Eigenlogistik) else it must be empty.

Show delivery date
--------------------

**Description**

If ll-logistic is used, show delivery date of simulation.

------------------------------------------------------------



Prices in Order Confirmation Mail
-----------------------------------

**Description**

Prices of the order response are used in the order confirmation mail if the customer has selected "delivery by LL logistic" else the prices form Team Beverage (stored in the shop) are used and the vat values for different rates and the total order sum has to be recalculated.

-------------------------------------------------------------

::

                    if ($oOrder->oxorder__bds_supplier->value == 'TB') {
                        $orderResponse = $this->replacePricesVatAndSumOfOrderResponse(
                            $orderResponse,
                            $oOrder->oxorder__bds_sapkundennummer->value
                        );
                    }

*Replace prices with Team Beverage prices and recalculate vat and total sum*

--------------------------------------------------------------

.. function:: replacePricesVatAndSumOfOrderResponse($orderResponse, $customerNumber)
	               
   :module: bds_erp.bds_erp_soapserver.php 

Replace prices with Team Beverage prices and recalculate vat and total sum
