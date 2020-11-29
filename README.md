# 분석 설계

## Event Storming 결과

MSAez를 사용한 event stroming결과

![캡처](https://user-images.githubusercontent.com/27837607/100529978-e9112a00-322f-11eb-9310-0043d188375b.JPG)



## 헥사고날 아키텍쳐


# 구현

## DDD 의 구현

MSAez로 구현한 Aggregate단위로 Entity를 선언 후, 구현 진행.

```
package shop;

import javax.persistence.*;
import org.springframework.beans.BeanUtils;
import shop.external.Payment;

import java.util.List;

@Entity
@Table(name="Order_table")
public class Order {

    @Id
    @GeneratedValue(strategy=GenerationType.AUTO)
    private Long id;
    private Long productId;
    private Long qty;
    private String orderStatus;
    private Long orderId;

    @PostPersist
    public void onPostPersist()
    {
        Ordered ordered = new Ordered();
        BeanUtils.copyProperties(this, ordered);
        ordered.setOrderStatus("Order");
        ordered.setOrderId(this.getOrderId());
        ordered.publishAfterCommit();

        shop.external.Payment payment = new shop.external.Payment();
        System.out.println("결제 이벤트 발생");
        payment.setId(this.getOrderId());
        payment.setStatus("Paid");
        OrderApplication.applicationContext.getBean(shop.external.PaymentService.class)
                .pay(payment);

    }
    @PreUpdate
    public void onPrePersist(){


        OrderCanceled orderCanceled = new OrderCanceled();
        BeanUtils.copyProperties(this, orderCanceled);
        if(orderCanceled.getOrderStatus().equals("Cancel"))
        {
            orderCanceled.setOrderStatus(this.getOrderStatus());
            orderCanceled.publishAfterCommit();
        }
        else
        {
            PaymentRequested paymentRequested = new PaymentRequested();
            BeanUtils.copyProperties(this, paymentRequested);
            paymentRequested.publishAfterCommit();



            //orderCanceled.publishAfterCommit();

            //Following code causes dependency to external APIs
            // it is NOT A GOOD PRACTICE. instead, Event-Policy mapping is recommended.

            //shop.external.Cancellation cancellation = new shop.external.Cancellation();
            // mappings goes here

            //OrderApplication.applicationContext.getBean(shop.external.CancellationService.class)
            //    .cancel(cancellation);
        }



    }


    public Long getId() {
        return id;
    }

    public void setId(Long id) {
        this.id = id;
    }
    public Long getProductId() {
        return productId;
    }

    public void setProductId(Long productId) {
        this.productId = productId;
    }
    public Long getQty() {
        return qty;
    }

    public void setQty(Long qty) {
        this.qty = qty;
    }

    public void setOrderStatus(String orderStatus){this.orderStatus = orderStatus;}
    public String getOrderStatus(){return orderStatus;}

    public Long getOrderId(){return orderId;}
    public void setOrderId(Long orderId){this.orderId = orderId;}
}

```
REST API에서의 테스트를 통하여 구현내용이 정상적으로 동작함을 확인.


## Request 방식의 아키텍쳐

## 이벤트 드리븐 아키텍쳐

## Gateway

## Poly Glot


# 운영

## SLA 준수

## CI / CD

## 유연성
