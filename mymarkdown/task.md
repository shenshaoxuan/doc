

# 数字菜单

### 新增菜单

* api

> api/v1/pos/keyboard?stringified=true

```json
{"name":"沈测试","code":"017","type":"MAIN","row":"6","col":"6","pageable":true,"key_width":1,"key_height":1,"key_bg_color":{"PRODUCT":"#ffffff","CATEGORY":"#ffffff","EMPTY":"#ffffff","ACTION":"#ffffff","ALERT":"#ffffff"},"key_font_color":{"PRODUCT":"#939393","CATEGORY":"#939393","EMPTY":"#939393","ACTION":"#939393","ALERT":"#939393"}}
```

* 在metadata中创建一条keyboard，版本号为1

### 新增菜单明细

* api

> api/v1/pos/keyboard/4446187662608465920/keys?auto_build_keys=true&stringified=true

```json
{
    "items":[
        {
            "x":0,
            "y":0,
            "width":1,
            "high":1,
            "type":"CATEGORY",
            "bg_color":"",
            "font_color":"",
            "status":"Y",
            "is_overflow":false,
            "name":"茶叶-过时",
            "code":"001",
            "object_id":"4219461197910507521",
            "alias":""
        }
    ],
    "section_order":1
}
```

* 循环items，生成poskey的model

* 在主档中查询主表信息,校验信息

* 从item中查找出属于商品种类的数据，存到字段作为key

  ```json
  {
    '4219461198292189185': {'products': [], 'category_children_ids': []}, 
   '4219461197910507521': {'products': [], 'category_children_ids': []}
  }
  ```

  

### 查询菜单详情

> api/v1/pos/keyboard/4446187662608465920?relation=all&is_new=true&include_state=true&include_pending_record=true&stringified=true

### 菜单明细

> api/v1/pos/keyboard/4446187662608465920/keys?section_order=1&include_total_section=true&stringified=true

# 订货收货流程(跳转)

## 门店紧急订货

* 选择门店，新增紧急订单，提交审核，生成订单号， 342101210099，分配配送中心：广州黄埔仓(猩米科技）

* 仓库核验，执行通过，生成要货单号，

* 打开订单查询菜单，可通过订货单号，查询到要货单，记录JDE单号(21000574)，或者要货单号(122101210008)

* 打开收货--配送收货，根据要货单号，或者JDE，搜寻到收货单(412101210008)，此时应该是已出库状态

* 打开明细，填写数量，保存提交

* api执行顺序：

  1. http://bohtest.heytea.com/api/v2/supply/receiving/4449817476900880384/confirm

  ```json
  {
      "confirmed_products":[
          {
              "id":"4449817476980572160",
              "product_id":"4437200540039938048",
              "received_quantity":20,
              "confirmed_quantity":"10.000",
              "confirmed_accounting_quantity":"10.000"
          },
          {
              "id":"4449817477135761408",
              "product_id":"4437200540832661504",
              "received_quantity":25,
              "confirmed_quantity":"15.000",
              "confirmed_accounting_quantity":"15.000"
          }
      ],
      "lan":"zh-CN"
  }
  返回ture或者false
  ```

  2. http://bohtest.heytea.com/api/v2/supply/receiving/4449817476900880384?stringified=true&lan=zh-CN

     重新查询单据

  3. http://bohtest.heytea.com/api/v2/supply/receiving/4449817476900880384/product?stringified=true&lan=zh-CN

     查询单据明细

# 单据增加门店第三层管理区域展示

## 修改表结构

```sql
alter table metadata_store add column branch_region_relation LONGTEXT not NULL;
```

## 更改的proto

```
// 第三层管理区域
string third_branch_region = 33;
```

* supply
  * receiving_diff_bi:配送收货差异， 单独取的仓库主档，(两个完成)测试完毕
  * receiving_bi: 收货单， 单独取的仓库主档，(两个完成),测试完毕
  * supply:退货单报表, 更改了sql，join了仓库主档，测试完毕
  * adjust: GetAdjustBiCollect,报废单报表,更改了sql，join了仓库主档, （需要改两个接口), 测试完毕

* report
  * Opretion_report：GetMaterialSupplies

## 触发写入任务

```shell
curl --location --request POST 'http://127.0.0.1:32446/pub?topic=supply.demand.batch.init' \
--header 'Authorization: Bearer B8uVlLdvOEmhtce1SZoz1Q' \
--header 'Content-Type: application/json' \
--data-raw '{"partner_id":"2","user_id":"4201196624908648450", "mode":"create"}'
```



# Seesaw订单明细查询增加字段

## go环境搭建

* 创建目录 xxx
* go mod init xxx =>>>>创建go.mod
* 创建main.go，如果其中有依赖，运行时会自动下载依赖，并将依赖写到go.mod,其实会将依赖下载到GOPATH下的pkg的mod文件夹下(但是这样无法避免多个项目使用不同版本的问题), go还会自动生成一个 go.sum 文件来记录 dependency tree
* go mod vendor:将包从mod中转移到项目下的vendor目录下

## 修改

* 在pkg/utils/下新增了包
* 更改了proto：grpc/proto/indent.proto
* 更改了grpc/format.go

## 需求：增加公司编号，公司名称

* company_id
* Company_code
* Company_name



## Cmd/base

```go
// Auto generate code. Thanks ['tianxuxin@126.com']
package cmd

import (
	"fmt"
	"os"
	"os/signal"

	"github.com/spf13/cobra"
	"hexcloud.cn/histore/supplier-platform/config"
	"hexcloud.cn/histore/supplier-platform/grpc/oauth"
	"hexcloud.cn/histore/supplier-platform/grpc/proto/delivery"
	entity "hexcloud.cn/histore/supplier-platform/grpc/proto/metadata"
	review "hexcloud.cn/histore/supplier-platform/grpc/proto/purchase_review"
	"hexcloud.cn/histore/supplier-platform/grpc/proto/receive"
	"hexcloud.cn/histore/supplier-platform/repo/database"
	"hexcloud.cn/histore/supplier-platform/server"
	"hexcloud.cn/histore/supplier-platform/service"
	"hexcloud.cn/histore/supplier-platform/utils"
)

func RunBase(cmd *cobra.Command, args []string) {
	// TODO Please start your server with non-blocking
	database.InitDB()
	if config.DefaultConfig.Flag.IsPlatForm {
		entity.Init()
		utils.InitUUID()
		delivery.Init()
		receive.Init()
		review.Init()
		oauth.Init()
		go service.AutoTask()
	}
	server.GrpcStart()
	go server.RunProxy()
	//go server.NsqStart()

	// end
	fmt.Println("All server model is started.")
	flag := make(chan os.Signal)
	signal.Notify(flag, os.Interrupt, os.Kill)
	<-flag
	// TODO Please stop your server and clean up
	server.NsqClose()
	server.GrpcStop()
	// end
	fmt.Println("\nStop server. Bye~")
}

```

## Cmd/root

```go
// Auto generate code. Thanks ['tianxuxin@126.com']
package cmd

import (
	"fmt"
	"os"

	"github.com/spf13/cobra"
	"hexcloud.cn/histore/supplier-platform/config"
	"hexcloud.cn/histore/supplier-platform/pkg/logger"
)

var rootCmd = &cobra.Command{
	Use:   "supplier-platform",
	Short: "",
	Run: func(cmd *cobra.Command, args []string) {
		RunBase(cmd, args)
		//cmd.Help()
	},
}

var baseCmd = &cobra.Command{
	Use:   "base",
	Short: "Start base server.",
	Run: func(cmd *cobra.Command, args []string) {
		RunBase(cmd, args)
	},
}

func Execute() {
	if err := rootCmd.Execute(); err != nil {
		fmt.Println(err)
		os.Exit(1)
	}
}

func init() {
	rootCmd.AddCommand(baseCmd)
	cobra.OnInitialize(onInitialize)
}

func onInitialize() {
	cfg := config.InitConfig()
	logger.InitLogger(cfg.Logger.Level, cfg.Logger.Env)
}

```

## config.yaml

```go
# Auto generate code. Thanks ['tianxuxin@126.com']
logger:
    level: info
    env: dev
grpc:
    port: 5001
db:
    dialect: mysql
    #connstr: xicha_stage:hex_xicha_ds123@tcp(rm-uf606j5022erh94it.mysql.rds.aliyuncs.com:3306)/supply_xicha_stage?parseTime=true&charset=utf8
    connstr: seesaw_boh_test:seesaw0617@tcp(rm-uf6xuw7c9aqu09c26so.mysql.rds.aliyuncs.com:3306)/seesaw_boh_test_receipt?charset=utf8&parseTime=true
    coststr: xicha_stage:hex_xicha_ds123@tcp(rm-uf606j5022erh94it.mysql.rds.aliyuncs.com:3306)/cost_xicha_stage?parseTime=True&loc=Local
    maxopen: 50
nsq:
    address: localhost:4150
    autoensuretopic: supplier_platform_auto_ensure
uuid:
    host: uuid
    port: 1234
flag:
    isplatform: true
externalpath:
    delivery: localhost:9089
    receive: localhost:8686
    review: localhost:9091
    metadata: localhost:5012
    oauth: localhost:9002
```

## Grpc/oauth

```go
package oauth

import (
	"context"
	fmt "fmt"
	math "math"

	grpc "google.golang.org/grpc"
	"google.golang.org/grpc/metadata"
	"hexcloud.cn/histore/supplier-platform/config"
	"hexcloud.cn/histore/supplier-platform/pkg/logger"
	"hexcloud.cn/histore/supplier-platform/utils"
)

var (
	DefaultClient HexOauthClient
	Cache         map[uint64]string = make(map[uint64]string)
)

func Init() {
	conn, err := grpc.Dial(config.DefaultConfig.ExternalPath.Oauth, grpc.WithInsecure(), grpc.WithDefaultCallOptions(grpc.MaxCallRecvMsgSize(math.MaxInt32), grpc.MaxCallSendMsgSize(math.MaxInt32)))
	if err != nil {
		fmt.Println("Init oauth conn faild.")
		panic(err)
	}

	DefaultClient = NewHexOauthClient(conn)
}

func GetUserName(ctx context.Context, id uint64) string {
	if v, ok := Cache[id]; ok {
		return v
	}

	u, p, s := utils.GetUidAndPidString(ctx)
	fresh := metadata.AppendToOutgoingContext(context.Background(), "user_id", u, "partner_id", p, "scope_id", s)
	user, err := DefaultClient.GetUserById(fresh, &GetUserRequest{Id: id})
	if err != nil {
		logger.Pre().Error("Get user info error.", err)
		Cache[id] = "Unknown"
		user.Name  = "Unknown"
	} else {
		Cache[id] = user.Name
	}
	return user.Name
}

```

## format

```go
package grpc

import (
	"context"
	"fmt"
	"strconv"
	"time"

	"github.com/golang/protobuf/ptypes"
	"github.com/golang/protobuf/ptypes/timestamp"
	"github.com/spf13/cast"
	"google.golang.org/grpc/metadata"
	pb "hexcloud.cn/histore/supplier-platform/grpc/proto"
	entity "hexcloud.cn/histore/supplier-platform/grpc/proto/metadata"
	"hexcloud.cn/histore/supplier-platform/model"
	utils "hexcloud.cn/histore/supplier-platform/pkg/utils"
)

// func ModelOrdersToProtoOrders(orders []*model.OrderWithMoney) (result []*pb.OrderWithMoney) {
// 	for _, item := range orders {
// 		result = append(result, ModelOrderToProtoOrder(item))
// 	}
// 	return
// }

// func ModelOrderToProtoOrder(order *model.OrderWithMoney) *pb.OrderWithMoney {
// 	return &pb.OrderWithMoney{
// 		Id:               order.Id,
// 		Code:             order.Code,
// 		StoreSecondaryId: order.StoreSecondaryId,
// 		StoreName:        order.StoreName,
// 		DistributeBy:     order.DistributeBy,
// 		OrderDate:        order.OrderDate,
// 		ArrivalDate:      order.ArrivalDate,
// 		SendDate:         order.SendDate,
// 		IsReceived:       order.IsReceived,
// 		DemandCode:       order.DemandCode,
// 		Money:            order.Money,
// 	}
// }

// func ProtoConditionToModelCondition(c *pb.ListOrderRequest) *model.OrderCondition {
// 	odf, _ := ptypes.Timestamp(c.OrderDateFrom)
// 	odt, _ := ptypes.Timestamp(c.OrderDateTo)
// 	adf, _ := ptypes.Timestamp(c.ArrivalDateFrom)
// 	adt, _ := ptypes.Timestamp(c.ArrivalDateTo)
// 	sdf, _ := ptypes.Timestamp(c.SendDateFrom)
// 	sdt, _ := ptypes.Timestamp(c.SendDateTo)
// 	return &model.OrderCondition{
// 		ReceiveBy:       c.ReceiveBy,
// 		DistributeBy:    c.DistributeBy,
// 		OrderDateFrom:   odf,
// 		OrderDateTo:     odt,
// 		ArrivalDateFrom: adf,
// 		ArrivalDateTo:   adt,
// 		SendDateFrom:    sdf,
// 		SendDateTo:      sdt,
// 		IsReceived:      c.IsReceived,
// 		DemandId:        c.DemandId,
// 		ProductId:       c.ProductId,
// 		Limit:           c.Limit,
// 		Offset:          c.Offset,
// 	}
// }

func ProtoConfirmSendRequestsToModelConfirmSend(c *pb.ConfirmSendRequests) (result []*model.ConfirmSend) {
	result = make([]*model.ConfirmSend, len(c.ConfirmSends))
	for i, item := range c.ConfirmSends {
		s, _ := ptypes.Timestamp(item.SendDate)
		a, _ := ptypes.Timestamp(item.ArriveDate)
		cs := &model.ConfirmSend{
			Id:         item.Id,
			SendDate:   s,
			ArriveDate: a,
			Details:    item.Details,
			AdjustImg:  item.Img,
			Type:       item.Type,
		}
		result[i] = cs
	}
	return
}

func ProtoCancelSendRequestsToModelCancelSend(c *pb.CancelSendRequests) (result []*model.CancelSend) {
	result = make([]*model.CancelSend, len(c.Ids))
	for i, item := range c.Ids {
		cs := &model.CancelSend{
			Id:     item,
			Reason: c.Reason,
		}
		result[i] = cs
	}
	return
}

func ProtoConfirmSendRequestToModelConfirmSend(c *pb.ConfirmSendRequest) (result *model.ConfirmSend) {
	return &model.ConfirmSend{
		Id:        c.Id,
		Details:   c.Details,
		AdjustImg: c.Img,
	}
}

func getFromAndToTime(t []*timestamp.Timestamp) (from, to time.Time) {
	if len(t) == 2 {
		from, _ = ptypes.Timestamp(t[0])
		from = time.Date(from.Year(), from.Month(), from.Day(), 0, 0, 0, 0, from.Location())
		from = from.Add(-8 * time.Hour)
		to, _ = ptypes.Timestamp(t[1])
		to = time.Date(to.Year(), to.Month(), to.Day(), 0, 0, 0, 0, to.Location())
		to = to.Add(24 * time.Hour)
		to = to.Add(-8 * time.Hour)
	}
	return
}

func ProtoOrderRequestToModelOrderCondition(c *pb.ListOrderRequest) *model.OrderCondition {
	odf, odt := getFromAndToTime(c.OrderDate)
	adf, adt := getFromAndToTime(c.ArrivalDate)
	sdf, sdt := getFromAndToTime(c.SendDate)
	rdf, rdt := getFromAndToTime(c.ReturnDate)
	return &model.OrderCondition{
		StoreId:         c.StoreId,
		SupplyId:        c.SupplyId,
		Status:          c.Status,
		Type:            c.Type,
		DemandId:        c.DemandId,
		JdeCode:         c.JdeCode,
		HwsCode:         c.HwsCode,
		OriginDemandId:  c.OriginDemandId,
		ProductId:       c.ProductId,
		OrderDateFrom:   odf,
		OrderDateTo:     odt,
		ArrivalDateFrom: adf,
		ArrivalDateTo:   adt,
		SendDateFrom:    sdf,
		SendDateTo:      sdt,
		ReturnDateFrom:  rdf,
		ReturnDateTo:    rdt,
		Limit:           c.Limit,
		Offset:          c.Offset,
		SortBy:          c.SortBy,
		Desc:            c.Desc,
	}
}

func ModelOrdersToProtoOrders(ctx context.Context, os []*model.Order) (result []*pb.Order) {
	var storeIds []uint64
	var resp *entity.ListEntityResponse
	var err error
	tempMap := map[string]byte{}
	for _, item := range os {
		l := len(tempMap)
		tempMap[item.StoreId] = 0
		if len(tempMap) != l {
			storeId, err := strconv.ParseUint(item.StoreId, 10, 64)
			if err != nil {
				panic(err)
			}
			storeIds = append(storeIds, storeId)
		}
	}

	//todo：从oauth取用户信息
	ctx = metadata.AppendToOutgoingContext(ctx, "partner_id", strconv.FormatUint(4183192445833445399, 10), "scope_id", strconv.FormatUint(0, 10), "user_id", strconv.FormatUint(4428480697887358977, 10))

	queryRequest := &entity.ListEntityRequest{
		SchemaName:   "STORE",
		Ids:          storeIds,
		ReturnFields: "id,name",
		Relation:     "company_info",
		Limit:        -1,
	}
	if resp, err = entity.DefaultClient.ListEntity(ctx, queryRequest); err != nil {
		return
	}
	fmt.Println(resp)
	var companyIds []uint64
	storeCompanyMap := make(map[uint64]interface{})
	for _, item := range resp.Rows {
		if etContent, err := utils.Struct2Map(item.Fields); err == nil {
			companyId := utils.GetUint64ArrayFromJSON(utils.GetMapFromJSON(etContent, "relation"), "company_info")
			companyIds = append(companyIds, companyId...)
			storeCompanyMap[item.Id] = companyId[0]
		}
	}
	companyMap := make(map[uint64]map[string]interface{})
	if len(companyIds) > 0 {
		queryRequest = &entity.ListEntityRequest{
			SchemaName:   "COMPANY_INFO",
			Ids:          companyIds,
			ReturnFields: "id,code,name",
			Limit:        -1,
		}
		if resp, err = entity.DefaultClient.ListEntity(ctx, queryRequest); err != nil {
			return
		}
		for _, item := range resp.Rows {
			if etContent, err := utils.Struct2Map(item.Fields); err == nil {
				companyMap[item.Id] = etContent
				fmt.Println(etContent)
			}
		}
	}

	for _, item := range os {
		od := &pb.Order{
			Id:              item.Id,
			Code:            item.Code,
			JdeCode:         item.JdeCode,
			HwsCode:         item.HwsCode,
			Type:            item.Type,
			StoreId:         item.StoreId,
			SupplyId:        item.SupplyId,
			Status:          item.Status,
			Price:           item.Price,
			ProductId:       item.ProductId,
			Quantity:        item.Quantity,
			RealQuantity:    item.RealQuantity,
			RecevedNum:      item.RecevedNum,
			RealRecevedNum:  item.RealRecevedNum,
			UnitId:          item.UnitId,
			OriginOrderCode: item.OriginOrderCode,
			AdjustImg:       item.AdjustImg,
			TaxRate:         item.TaxRate,
			Reason:          item.Reason,
		}
		pid := cast.ToUint64(item.StoreId)
		companyId := storeCompanyMap[pid].(uint64)
		companyInfo := companyMap[companyId]
		od.CompanyId = companyId
		od.CompanyCode = companyInfo["code"].(string)
		od.CompanyName = companyInfo["name"].(string)

		if !item.OrderDate.IsZero() {
			od.OrderDate = item.OrderDate.Format(time.RFC3339)
		}
		if item.Type == "要货单" && item.Status != "INITED" && item.Status != "PENDING" && !item.ArrivalDate.IsZero() {
			od.ArrivalDate = item.ArrivalDate.Format(time.RFC3339)
		}
		if item.Type == "要货单" && item.Status != "INITED" && !item.SendDate.IsZero() {
			od.SendDate = item.SendDate.Format(time.RFC3339)
		}
		if !item.ReturnDate.IsZero() {
			od.ReturnDate = item.ReturnDate.Format(time.RFC3339)
		}
		if item.Type == "要货单" && !item.ExpectArrivalDate.IsZero() {
			od.ExpectArrivalDate = item.ExpectArrivalDate.Format(time.RFC3339)
		}
		if !item.UpdatedAt.IsZero() {
			od.UpdatedAt = item.UpdatedAt.Format(time.RFC3339)
		}
		//if item.UpdatedBy != 0 {
		//	od.UpdatedBy = oauth.GetUserName(ctx, item.UpdatedBy)
		//}
		result = append(result, od)
	}
	return
}

func ProtoConfirmEnsureRequestToModelConfirmEnsure(ceq *pb.ConfirmEnsureRequest) *model.ConfirmEnsure {
	return &model.ConfirmEnsure{
		Type: ceq.Type,
		Ids:  ceq.Ids,
	}
}

// func ModelOrdersToProtoOrdersDetails(orders []*model.OrderWithProductPrice) (result []*pb.OrderWithMoney) {
// 	for _, item := range orders {
// 		result = append(result, ModelOrderWithProductPriceToProtoOrder(item))
// 	}
// 	return
// }

// func ModelOrderWithProductPriceToProtoOrder(order *model.OrderWithProductPrice) *pb.OrderWithMoney {
// 	return &pb.OrderWithMoney{
// 		Id:               order.Id,
// 		Code:             order.Code,
// 		StoreSecondaryId: order.StoreSecondaryId,
// 		StoreName:        order.StoreName,
// 		DistributeBy:     order.DistributeBy,
// 		OrderDate:        order.OrderDate,
// 		ArrivalDate:      order.ArrivalDate,
// 		SendDate:         order.SendDate,
// 		IsReceived:       order.IsReceived,
// 		DemandCode:       order.DemandCode,
// 		ProductId:        order.ProductId,
// 		Quantity:         order.Quantity,
// 		UnitId:           order.UnitId,
// 		Money:            order.Price,
// 	}
// }

```

Api:http://seesaw-boh.hexcloud.cn/api/v1/supplier-platform/order/query

查找所调用方法：/proto/reveive/indent.proto下的SupplyIndentService.QueryOrder

更改proto：

```go
message Order {
    uint64 id                  = 1;     // 单据 id
    string code                = 2;     // 要货单编号
    string jde_code            = 3;     // JDE 单号
    string hws_code            = 4;     // HWS 单号
    string type                = 5;     // 单据类型
    string store_id            = 6;     // 门店编号
    uint64 supply_id           = 7;     // 供应商 id
    string order_date          = 8;     // 要货日期
    string arrival_date        = 9;     // 到货日期
    string send_date           = 10;    // 发货日期
    string return_date         = 11;    // 退货日期
    string status              = 12;    // 状态
    double price               = 13;    // 价格
    uint64 product_id          = 14;    // 商品 id
    double quantity            = 15;    // 商品数量
    double real_quantity       = 16;    // 实际商品数量
    uint64 unit_id             = 17;    // 商品单位
    string origin_order_code   = 18;    // 原单据编号
    double receved_num         = 19;    // 收货数量
    double real_receved_num    = 20;    // 实际收货数量
    string adjust_img          = 21;    // 调整单图片
    string expect_arrival_date = 22;    // 预估到货日期
    string updated_by          = 23;    // 更新人
    string updated_at          = 24;    // 更新时间
    double tax_rate            = 25;    // 税率
    string remark              = 26;    // 备注
    string withdraw_at         = 27;    // 测回时间
    string withdraw_reason     = 28;    // 撤回原因
    string reason              = 29;    // 驳回原因
    uint64 company_id          = 30;    // 公司id
    string company_code        = 31;    // 公司code
    string company_name        = 32;    // 公司名称
}
```

编译proto：

```
protoc  -I$GOPATH/src/github.com/grpc-ecosystem/grpc-gateway/third_party/googleapis \--go_out=plugins=grpc,paths=source_relative:. --validate_out="lang=go,paths=source_relative:." indent.proto
```

```
--go_out=Mgoogle/api/annotations.proto=github.com/grpc-ecosystem/grpc-gateway/third_party/googleapis/google/api,plugins=grpc:.
```

 移动pkg/utils

代码结构：

* grpc：下的go文件，是方法入口， indent.go
* service：处理基本的逻辑， platform.go
* repo/database: 数据库逻辑处理， indent_platform.go



# 订货预估

根据销售量，消耗量  预估日订货量

```
string third_branch_region = 29;
```

根据历史数据，评估订货数量 



影响因素：

* 门店自己更改销售预估
* 手动订货
* 库存盘点

aaa

* 预估营业额：销售营业额 = 净利润 + 退款 + 折扣额
* 预估销售占比
* 用户调整营业额，需要反向计算预估销量
* 通过bom分解，计算订货量
* 订货通过历史数据计算或者通过营业的销售占比反推，考虑零售商品，无BOM的计算
* 考虑报损商品所占的毛额
* 在途：在订货周期内未收货和未到店的（收货单和调拨单）



# 春节假期问题

深圳湾万象城，报损每天都会保存失败

门店的反馈，报损页面停留超过五分钟了再电脑上保存报损单，可能会提示报损失败，然后报损的都为0

失败日志：

umarshal error fail: Exception calling application: (raised as a result of Query-invoked autoflush; consider using a session.no_autoflush block if this flush is occurring prematurely) (MySQLdb._exceptions.IntegrityError) (1062, "Duplicate entry '4459220566650617857-4294354923163877376-01' for key 'supply_adjust_product_unique_index'") [SQL: 'INSERT INTO supply_adjust_product (created_at, updated_at, id, partner_id, user_id, adjust_id, product_id, product_code, product_name, unit_id, unit_name, unit_spec, accounting_unit_id, accounting_unit_name, accounting_unit_spec, quantity, accounting_quantity, confirmed_quantity, is_confirmed, item_number, material_number, adjust_store, adjust_date, reason_type, convert_accounting_quantity, is_bom) VALUES (%s, %s, %s, %s, %s, %s, %s, %s, %s, %s, %s, %s, %s, %s, %s, %s, %s, %s, %s, %s, %s, %s, %s, %s, %s, %s)'] [parameters: ((datetime.datetime(2021, 2, 17, 0, 9, 14, 509406), datetime.datetime(2021, 2, 17, 0, 9, 14, 512504), 4459426603895717888, 4206763836826861569, 4231712376929796168, 4459220566650617857, 4294354923163877376, '30990058', '紫米紫薯米面包', 4219460847195389953, '个', 'EA', 4219460847195389953, '个', 'EA', 9.0, 9.0, None, 0, 1, '30990058', 4219460804656758785, datetime.datetime(2021, 2, 16, 0, 0), '01', None, 0), (datetime.datetime(2021, 2, 17, 0, 9, 14, 511622), datetime.datetime(2021, 2, 17, 0, 9, 14, 512511), 4459426603904139264, 4206763836826861569, 4231712376929796168, 4459220566650617857, 4294354923465768960, '30990059', '芋泥咸蛋黄米面包', 4219460847195389953, '个', 'EA', 4219460847195389953, '个', 'EA', 6.0, 6.0, None, 0, 2, '30990059', 4219460804656758785, datetime.datetime(2021, 2, 16, 0, 0), '01', None, 0))] (Background on this error at: http://sqlalche.me/e/gkpj)

## 新增报废单接口

http://bohtest.heytea.com/api/v2/supply/adjust/create

## 主从保存做法





# 销售配方物料拆解异常报表

## 测试数据

```sql
select e.pos_order_id,
e.sequence_id, 
pc."id" as product_id, 
pc.code as product_code, 
pc.name as product_name, 
sc."id" as store_id, 
sc.code as store_code, 
sc.name as store_name, 
b.error from (
select bom_explain_ret,
  e_ticket_id,
             bom_explain_ret::json #>> '{response,0,error}'      as error,
             bom_explain_ret::json #>> '{response,0,store_id}'   as store_id,
             bom_explain_ret::json #>> '{response,0,product_id}' as product_id
      from sales_inventory_detail
  where updated > 1609868607 and bom_explain_ret::json #>> '{response,0,error}' is not null
   and bom_explain_ret::json #>> '{response,0,error}' like '%no_bom%') b 
   inner JOIN product pc on pc."id" = b.product_id 
   inner join store sc on sc."id" = b.store_id
   inner join e_ticket e on e."id" = b.e_ticket_id
```

```
label:not_match_label_group;material:no_inventory:333327176646589183;material:no_inventory:333327177804217087;material:no_inventory:333327193105038079;material:no_inventory:333327245852605183;material:no_inventory:333327246607579903;material:no_inventory:333327254039886591;material:no_inventory:333327359686015743



material:no_inventory:333327176646589183;material:no_inventory:333327177804217087;material:no_inventory:333327193105038079;material:no_inventory:333327245852605183;material:no_inventory:333327246607579903;material:no_inventory:333327254039886591;material:no_inventory:333327359686015743



no_bom
```

## 需求

> 维度：按门店按单按商品—详情到配方物料
>
> 异常类型1：商品无配方的情况（过滤零售物料后）——按门店按订单按品
> 异常原因：—列表新增字段
> ① 商品无对应配方—爆出商品，写入异常描述
> ② 门店与商品配方是没有对应关系—爆出商品和门店，写入异常描述
>
> 异常类型2：走兜底逻辑的拆解——按门店按单品维度
> 异常原因：
> ① 标签不匹配直接兜底--爆出原标签组和拆解的标签组，写入异常描述
> ② 标签重复去重后兜底—爆出重复标签、原标签组和拆解的标签组，写入异常描述
> ③ 同一可选项下有多个标签，丢弃一个后兜底-爆出同组标签、原标签组和拆解的标签组，写入异常描述
>
> 异常类型3：无库存不拆解——按门店按单按品按配方物料
> 异常原因：
> ① 配方物料下原物料无库存—暴出商品和配方物料，写入异常描述

## 未完成的点

* 商品无配方的

  * 过滤零售物料 完成
  * 商品无对应配方,门店与商品配方是没有对应关系,如何区别？不需要区别
  * 异常类型如何写入 完成
  * 异常描述如何写入？完成
* 无库存不拆解

  * 异常类型如何写入 完成
  * 异常描述如何写 完成
* 无标签的未确定数据结构 -- 未完成
* 先按照营业日期倒序，再按照交易时间倒序 -- 未完成
* bom_material插入一些数据做测试，bom_material从何同步而来？ -- 完成，需要配置环境变量
* 无库存的对应不上物料编号，应该使用BOH的物料编号，而不是主档中心的编号 --完成，不需要修改
* 导出无法使用 --完成
* 异常类型保持和前端展示的一样 --完成

## 开发

* 增加接口：/Users/shenshaoxuan/hekuo/report/report/api/operation_report.py----GetSalesExplainedErrorFromMetadataReport

## 部署

* 分支：feature/20210207_sales_exception_report

* 新增环境配置,正式环境需要配置数据库环境变量，并且需要注意pg库的账号权限，需要读取另外一个库bom的数据

```yaml
postgre:
    user: xicha_stage
    password: hex@xicha_stage123
    host: pgm-uf67v0i92o88o8d4.pg.rds.aliyuncs.com
    port: 3433
    database: report_xicha_stage

bom_postgre:
    user: xicha_stage
    password: hex@xicha_stage123
    host: pgm-uf65x0c3ex0563sg.pg.rds.aliyuncs.com
    port: 1921
    database: venom_xicha_stage
```

* 同步数据，比如product_cache，需要过滤掉零售商品
  * 导入环境变量：import start
  * 设置table类属性：__table_args__ = {'extend_existing': True}
  * 新增参数：ProductCache.build(partner_id=2, user_id=4201196624908648450)

* gateway编译
* 注意往master合并时可能因为proto文件冲突



# pos的user下发排序

* 后端已排序，需要林炳强确认给前端数据的排序方式

  

# 建议订货量

## task

* **task1**:验证物料分解T-1是否写入成功，3.3日验证3.2的已写入成功

* 系统根据历史数据计算预估营业额，不是预估总额，而是每件商品的销售额
  * 前端参数：store_id
  * 筛选门店，取出前一周同比天的毛额 * 50%
  * 计算当前日期，并推算前四周的日期，按照日期分组，将毛额sum，并*50%， case某天无数据，则取前一周的100%
  * 将两个结果按照store_id和product_id，group并且sum营业额

```
前置任务: 将物料分解报表的固化时间由t-2调整至t-1, 发布后手动固化一次昨天的数据即可 0.5d
mrp的api时间预估::
(1). 固化流程中间表结构设计和索引设计 
(2).业务数据表supply_suggest/supply_suggest_product两个表的结构设计和索引设计
(3). 固化自动任务, 即业务数据的计算
人天: (1) + (2) + (3): 3~3.5d
(4). 固化任务源数据同步: 从生产同步(单品销售报表至少1个月的数据) 物料分解报表 至少4-7天的数据, 报废数据也从生产同步4-7天的数据  1~2d
(5) 反馈自动任务: 盘点后计算建议订货量 (取数逻辑相对变更, 例如千元结构等取前4天数据的, 都有变动) -- 后台计算, 不展示 2d

http接口:
1. 预估营业额和预估销量查询接口 1.5d
2. 修改预估销量/预估营业额的接口 2d 
3. 实时重算建议订货量的接口 1.5d 
```

![image-20210302134255474](/Users/shenshaoxuan/Library/Application Support/typora-user-images/image-20210302134255474.png)



## supply增加的文件

### config/supply_local.yaml

增加了report数据库的连接方式

* 不能配置两个关系数据库的限制，需要将配置的名称修改一下

### supply/model/suggest.py

新增了model

* SupplySuggestBatch
* SupplySuggestStoreBatch
* 

### supply/module/suggest.py

增加了方法

### supply/task/suggest.py

增加了自动任务

*  创建批次，创建门店批次
* 开始计算订货量
* 

### supply/utils/helper.py

增加了自动任务触发的方式的变量

## 自动任务

### 1. 建议预估初始化

##### 接收参数

* partner_id
* user_id
* demand_date：将当天日期加一天，每日计算的应该是下日的
* batch_type：批次类型，必传

##### 流程

* 查看当天日期是否已经计算过(查询时没加 partner_id)，如果有，则直接return
  * 创建SupplySuggestBatch
  * 查找所有门店信息(条件是open状态，并且可用的)
  * 创建SupplySuggestStoreBatch，全部插入
  * 循环SupplySuggestStoreBatch， 循环发送消息supply.suggest，开始预估

### 2. 预估计算建议订货量

* 查找第二层信息，如果没有数据则return，否则开始创建第三层

* 获取各种计算因子，有默认值

* 获取前四周预估总营业额,从report的product_sales_report获取数据，获取到四个dict

  * 注意是否存在某天歇业未营业，导致没有任何商品销售
  * 未传入partner_id
  * 获取到每件商品的毛营业额，净额+折扣额度

  ```python
  {
    "product_id":SUM(net_amount)+SUM(discount_amount)
  }
  ```

  

* 获取前四天的产品销量，用来计算千元销售结构:form_dict

  * 未传入partner_id

```json
{
  "pruduct_id":{"quantity": 23.12} //product_id和每件商品过去四天的平均销量
}
```



* 获取前四天的产品报损数量
  * 未传入partner_id

```json
{
  "pruduct_id":[unit_id, quantity] //product_id和每件商品过去四天的报损单位和平均报损
}
```



* 获取前四天的物料分解：

```json
{
  "product_id": [
    {"bom_pid":"bom_id1", "bom_qty": bom_quantity1}, 
    {"bom_pid":"bom_id2", "bom_qty": bom_quantity2}, 
    ........
  ], 
  .....
}
```

* 从前四天同比天的四个dict中，拿到所有的product_id，添加到product_list，等于说能写到预估表里的，只能是从最近四周的同比天销售过的
* 计算预估的营业额：计算规则，如果此产品过去四周每日都有销售，则按规则计算营业额，否则条件不符合，直接取最近的一天数据营业额，将所有营业额累加，得到实际总营业额

* 汇总全部销量商品：从总营业额的四个dict中，获取所有的product_id
* 取出product的类型：product_dict =====>>>>> {"product_id", (product_type, bom_type)}
* 计算预估营业总额，过去四周所有商品的销售额，按照规则计算====>>>>> 
* 循环product_list
  * 根据product最近四天的销售总量/过去四周总计销售额====>>>>> 得到计算千元结构数
  * 按照规则计算每件商品的预估营业额，根据千元结构数反推====>>>>>产品预估销量
  * 产品预估销量 =  （产品预估销量  + 报废数量）
  * 将以上信息append，如果有bom信息，则appendbom的 信息
  * 表model：supply_suggest，预备插入
    * store_id：门店id
    * product_id：产品id
    * sys_amount:预估总营业额
    * thousand_form：千元结构
    * sys_quantity：产品预估销量
    * adjust_qty：报废数量
  * 表supply_suggest_product，如果商品有分解，预备插入
    * 如果商品的product_type是’’成品‘‘和’‘半成品’‘的，并且bom_type是’制造‘类型的，获取一个产品的报损数量
    * store_id
    * suggest_id
    * product_id
    * bom_product_id
    * avg_qty
    * onway_qty
    * adjust_qty
* 将数据全部插入

* 发送消息计算建议门店的订货量：{store_id, demand_date, partner_id, }

问题

* 创建第三层时有一些查询
* 总营业额的计算方式是累加的，这个需要再确认一遍
* 产品销售预估量的计算方式：这里应该乘以系统预估营业额，而非实际营业额
* 注意sql查询的性能，能一次不多次
* 查询产品销量时，是否存在某天未营业无销量的现象，如果存在则计算不准确x

### 3. 开始计算

查询出需要计算建议订货量的商品

获取在途商品

实时库存获取，如果配送类型是bread：加入oem_ids

### 新增接口

#### 查询预估列表

* 查询表：supply_suggest

* 条件
  * 门店：store_id
  * 日期：大于等于当天的所有预估数据

* 查询当前门店下的所有商品(已写入建议订货量表的和当天新品)

  ***当天新品的取法：与昨日相比，新增的销售商品，这个应该取的是实时数据吧？product_sales_record***

  * 如果用户修改过预估销量，则返回用户输入的预估销量，否则返回系统预估销量
  * 如果用户修改过预估营业额，则返回用户输入的预估营业额，否则返回系统预估营业额

* 根据查询出的商品获取一个最长的订货周期:

  * 当天订货单，demand_product

* 根据周期，将所有数据整合成以下结构

  ```json
  [
      {
          "id":123,
          "product_id":1,
          "product_name":"xx",
          "pruduct_type":"xx",
          "forecast_data":[
              {
                  "date":"2021-3-4",
                  "real_amount":1000,
                  "forecast_amount":2000,
                  "forecast_quantity":20,
                  "thousand_form":8.2
              },
              {
                  "date":"2021-3-5",
                  "real_amount":1000,
                  "forecast_amount":2000,
                  "forecast_quantity":20,
                  "thousand_form":8.2
              },
              {
                  "date":"2021-3-6",
                  "real_amount":1000,
                  "forecast_amount":2000,
                  "forecast_quantity":20,
                  "thousand_form":8.2
              }
          ]
      }
  ]
  
  ```

  #### 保存预估列表

  * 用户更改预估营业额

  ```
  后台根据千元结构，将所有的商品预估销量重算一遍
  更新supply_suggest：填入pre_quantity，pre_amount，amt_changed
  更新supply_suggest_product
  写日志：supply_suggest_log
  ```

  

  * 用户更改预估销量

  ```
  已预估的商品：
  更新：pre_quantity，pre_reason， 重算pre_amount，qty_changed 
  未预估的商品(新上架的商品)
  ```

  

  ```
  '''
          product_id	demand_date	quantity
          第一步先处理数据：做一个map
          key：（product_id， date）， value:value
          第二步，循环product_ids，
              循环日期：
                  []
                  如果(product,date) in map
                  组装数据，然后加入到外层的list当中
              否则:
                  将数据全置为0
          最后给这个list排序，按照每个字典中的code
          第三步：处理新商品
              查出新商品的price
              循环新商品数据，循环日期，组装数据，将数据append的一个列表中，然后排序
          第四部：
              将新商品extend到老商品的list当中
          '''
          
   """
  select max(B.arrival_days)
  from supply_demand A
  join supply_demand_product B on B.demand_id=A.id
  where A.receive_by=4219460795467038721 and A.demand_date='2020-09-19 00:00:00' and A.partner_id=2
  """
  
  """
  SELECT psr.product_id, pc.code, pc.name, pc.product_type
  FROM (
  		SELECT DISTINCT psr_a.product_id 
  		FROM product_sales_record psr_a
  		WHERE
  			psr_a.store_id = 4271939601733517313 
  			AND psr_a.operation_date = 1614873600 
  			AND NOT EXISTS (
  			SELECT 1 
  			FROM product_sales_record psr_b
  			WHERE
  				psr_b.store_id = 4271939601733517313 
  				AND psr_b.operation_date = 1614787200 
  				AND psr_b.product_id = psr_a.product_id 
  			) 
  	) psr
  JOIN product_cache pc on pc.id=psr.product_id
  """
         
  ```

  



## 查询建议订货量

## 保存用户预估

* 新商品给个标识, 新商品返回商品单价
* 新商品，保存，直接查询单价，然后累计营业额
* 如果用户修改营业额，需要将新商品抛出，然后重新计算销量

## 查询订货单的建议订货量

# 新增接口

1. 合阔云端服务提供openAPI   （GET，oauth认证，固定token)    （喜茶希望自己拉取版本信息）
2、version list：喜茶提供版本信息，合阔云端服务提供ADD接口，将信息写入到合阔的信息表中（consul) 

3、在线列表版本上报：合阔云端服务提供接口 上报目前该门店的使用版本号[ 喜茶提供门店第二编号,类似我们POS 机的在线列表的信息】





1. 参数,store_id, app_name
   * 主档获取当前版本===>>>version
2. 参数：app_name， version， oss地址
   * 写到consul

3. 



# pos菜单不一致

取缓存

```
hex:4206763836826861569:0:pos_kb_keys:4353130198439297025
```



通过pos找到门店，查看门店的自定义菜单

然后修改这个菜单，保存

查看菜单缓存是否有改动，来确定保存后是否build缓存

如果有改变，则查看是哪个位置build了缓存



经查看，是通过自动任务创建的缓存

* 不能通过判断section_id=0，来过滤数据，因为某个大类自动带入的数据确实是没有section_id和section_order
* 如果用户没有保存过，则section_order和section_id全为0，不存在只有一个的情况

经过排查：

确定唯一的解决办法是修改保存接口，将数据清除掉

查找section_id和section_order为何会为0

前端传入参数：

​	section_id：1234

​	parent_id：123

后台：

​	如果当前页面已经保存过商品，则条件为delete(keyboard=123, section_id=123, id not in (ids), scope_id=1)

​	如果用户没保存过，直接通过商品大类带入的，则以上这种删除方式无法删除

​	新做法：

​		delete(keyboard=123,, id not in (ids),id in (parent_id.childs), section_Id<>0  , scope_id=1)---->>>这种情况可能会删除掉用户已经在其他页保存的数据

​		删除掉数据，并且不能影响当前parent_id下的已经保存的数据



例如1：删除未保存的第二页的A商品，如果第一页用户保存过A商品，则不能影响第一页

​	第一页的A商品一定是有坐标数据和section_id的



例如2：删除已保存的第二页的A商品，如果第一页用户保存过A商品，则不能影响第一页

​	第一页的A商品一定是有坐标数据和section_id的











