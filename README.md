# cityChoose
地址选择的三级联动，支持外部传入地址来确定当前位置。


//
//  CityChooseViewController.h
//  yoyo
//
//  Created by YoYo on 16/5/12.
//  Copyright © 2016年 cn.yoyoy.mw. All rights reserved.
//

#import <UIKit/UIKit.h>
typedef void(^CityBlock)(NSString *province, NSString *city, NSString *area); //选择地址后的回调block
@interface CityChooseViewController : UIViewController

@property (copy, nonatomic) NSString *province; //选中的省
@property (copy, nonatomic)NSString *city;//选中的市
@property (copy, nonatomic) NSString *area; //选中的地区


@property (copy, nonatomic) CityBlock cityInfo; //选择的城市信息
- (void)returnCityInfo:(CityBlock)block; //赋值的时候回调

@end


//
//  CityChooseViewController.m
//  yoyo
//
//  Created by YoYo on 16/5/12.
//  Copyright © 2016年 cn.yoyoy.mw. All rights reserved.
//

#import "CityChooseViewController.h"

#define screen_width [UIScreen mainScreen].bounds.size.width
#define screen_height [UIScreen mainScreen].bounds.size.height

@interface CityChooseViewController ()<UITableViewDelegate, UITableViewDataSource>

@property (strong,nonatomic)UILabel *addressL;

@property (strong, nonatomic) UITableView *mainTableView; //省
@property (strong, nonatomic) UITableView *subTableView; //市
@property (strong, nonatomic)UITableView *areaTableView; //区

@property (strong, nonatomic)NSDictionary *provinceDic;//省份字典
@property (strong, nonatomic)NSDictionary *cityDic;//市字典

@property (strong, nonatomic) NSArray *provinceList; //省份列表
@property (strong, nonatomic) NSArray *cityList;//城市列表
@property (strong, nonatomic) NSArray *arealist;//区列表

@property (assign, nonatomic) NSInteger selprovinceIndex;//当前选择的省份下标
@property (assign, nonatomic) NSInteger selcityIndex;//当前选择的市下标
@property (assign, nonatomic) NSInteger selAreaIndex;//当前选择的区下标


@property (assign, nonatomic) BOOL clickRefresh;//是否是点击主列表刷新子列表,系统刚开始默认为NO


@property (strong, nonatomic) UIButton *sureBtn;//push过来的时候，右上角的确定按钮

@end

@implementation CityChooseViewController

- (void)viewDidLoad {
    [super viewDidLoad];
    if (!_province.length) {
        _province = @"广东省";
        _city = @"广州市";
        _area = @"天河区";
    }
    self.title = [NSString stringWithFormat:@"%@ %@ %@",_province,_city,_area];
    [self addTableView];
    [self initNavigationBar];
}

- (void)initNavigationBar
{
    self.navigationItem.rightBarButtonItem = [[UIBarButtonItem alloc]initWithTitle:@"确定" style:UIBarButtonItemStylePlain target:self action:@selector(sureAction:)];
}

//赋值
- (void)returnCityInfo:(CityBlock)block {
    _cityInfo = block;
}

#pragma mark 创建两个tableView
- (void)addTableView {
    
    self.view.backgroundColor = [UIColor whiteColor];
    //获取目录下的city.plist文件
    NSString *plistPath = [[NSBundle mainBundle] pathForResource:@"Address" ofType:@"plist"];
    _provinceDic = [NSDictionary dictionaryWithContentsOfFile:plistPath];
    
    //获得省份列表
    _provinceList = [_provinceDic allKeys];
    _selprovinceIndex = [_provinceList indexOfObject:_province];

    //获得城市列表
    _cityDic = [_provinceDic objectForKey:_province][0];
    _cityList = [_cityDic allKeys];
    
    _selcityIndex = [_cityList indexOfObject:_city];
    
    //获得区列表
    _arealist = [_cityDic objectForKey:_city];
    _selAreaIndex = [_arealist indexOfObject:_area];
    
    //tableView
    _mainTableView = [[UITableView alloc] initWithFrame:CGRectMake(0, 64, screen_width / 3 + 1, screen_height - 100) style:UITableViewStylePlain];
    _mainTableView.dataSource = self;
    _mainTableView.delegate = self;
    [_mainTableView selectRowAtIndexPath:[NSIndexPath indexPathForRow:_selprovinceIndex inSection:0] animated:YES scrollPosition:UITableViewScrollPositionMiddle]; //默认省份选中第一行
    
    [_mainTableView scrollToRowAtIndexPath:[NSIndexPath indexPathForRow:_selprovinceIndex inSection:0] atScrollPosition:UITableViewScrollPositionMiddle animated:YES];//滚动到那一行
    
    [self.view addSubview:_mainTableView];
    
    _subTableView = [[UITableView alloc] initWithFrame:CGRectMake(screen_width / 3, 64, screen_width * 1 / 3, screen_height - 100) style:UITableViewStylePlain];
    _subTableView.dataSource = self;
    _subTableView.delegate = self;
    [_subTableView scrollToRowAtIndexPath:[NSIndexPath indexPathForRow:_selcityIndex inSection:0] atScrollPosition:UITableViewScrollPositionMiddle animated:YES];//滚动到那一行
    [_subTableView selectRowAtIndexPath:[NSIndexPath indexPathForRow:_selcityIndex inSection:0] animated:YES scrollPosition:UITableViewScrollPositionMiddle];
    
    
    [self.view addSubview:_subTableView];
    
    _areaTableView = [[UITableView alloc] initWithFrame:CGRectMake(screen_width  * 2/ 3, 64, screen_width * 1 / 3, screen_height - 100) style:UITableViewStylePlain];
    _areaTableView.dataSource = self;
    _areaTableView.delegate = self;
    [_areaTableView scrollToRowAtIndexPath:[NSIndexPath indexPathForRow:_selAreaIndex inSection:0] atScrollPosition:UITableViewScrollPositionMiddle animated:YES];//滚动到那一行
    [_areaTableView selectRowAtIndexPath:[NSIndexPath indexPathForRow:_selAreaIndex inSection:0] animated:YES scrollPosition:UITableViewScrollPositionMiddle];
    [self.view addSubview:_areaTableView];
    
    
    
}
#pragma mark 确认选择
-(void) sureAction:(UIBarButtonItem *)item {
    _cityInfo(_province,_city, _area);
    [self.navigationController popViewControllerAnimated:YES];
}

-(NSInteger)numberOfSectionsInTableView:(UITableView *)tableView {
    return 1;
}
-(NSInteger)tableView:(UITableView *)tableView numberOfRowsInSection:(NSInteger)section {
    if ([tableView isEqual:_mainTableView]) {
        return _provinceList.count;
    }
    else if ([tableView isEqual:_subTableView])
    {
        return _cityList.count;
    }
    else
    {
        return _arealist.count;
    }
}

-(UITableViewCell *)tableView:(UITableView *)tableView cellForRowAtIndexPath:(NSIndexPath *)indexPath {
    if ([tableView isEqual:_mainTableView]) {
        static NSString *mainCellId = @"mainCellId";
        UITableViewCell *cell = [tableView cellForRowAtIndexPath:indexPath];
        if (cell == nil) {
            cell = [[UITableViewCell alloc] initWithStyle:UITableViewCellStyleDefault reuseIdentifier:mainCellId];
        }
        if (_selprovinceIndex == indexPath.row)
        {
            cell.accessoryType = UITableViewCellAccessoryCheckmark;
        }
        cell.textLabel.text = _provinceList[indexPath.row];
        cell.textLabel.font = [UIFont systemFontOfSize:12];
        return cell;
    }
    else if ([tableView isEqual:_subTableView])
    {
        static NSString *subCellId = @"subCellId";
        UITableViewCell *cell = [tableView cellForRowAtIndexPath:indexPath];
        if (cell == nil) {
            cell = [[UITableViewCell alloc] initWithStyle:UITableViewCellStyleDefault reuseIdentifier:subCellId];
        }
        if (_selcityIndex == indexPath.row)
        {
            cell.accessoryType = UITableViewCellAccessoryCheckmark;
        }
        cell.textLabel.text = _cityList[indexPath.row];
        cell.textLabel.font = [UIFont systemFontOfSize:12];

        return cell;
    }
    else
    {
        static NSString *subCellId = @"areaCellId";
        UITableViewCell *cell = [tableView cellForRowAtIndexPath:indexPath];
        if (cell == nil) {
            cell = [[UITableViewCell alloc] initWithStyle:UITableViewCellStyleDefault reuseIdentifier:subCellId];
        }
        if (_selAreaIndex == indexPath.row)
        {
            cell.accessoryType = UITableViewCellAccessoryCheckmark;
        }

        cell.textLabel.text = _arealist[indexPath.row];
        cell.textLabel.font = [UIFont systemFontOfSize:12];
        return cell;
    }
}


-(void)tableView:(UITableView *)tableView didSelectRowAtIndexPath:(NSIndexPath *)indexPath {
    //选择后当前cell的状态变为select 其他cell变为不select
    UITableViewCell *cell = [tableView cellForRowAtIndexPath:indexPath];
    cell.accessoryType = UITableViewCellAccessoryCheckmark;
    
    if ([tableView isEqual:_mainTableView]) {
        _province = _provinceList[indexPath.row]; //赋值
        if (_selprovinceIndex != indexPath.row)//选择的省份有变化
        {
            _selcityIndex = 0;
            _selAreaIndex = 0;
            //刷新市、区
            _cityDic = [_provinceDic objectForKey:_province][0];
            _cityList = [_cityDic allKeys];
            _city = _cityList.firstObject;
            _selcityIndex = 0;
            //获得区列表
            _arealist = [_cityDic objectForKey:_city];
            _area = _arealist[_selAreaIndex];
            [_subTableView reloadData];
            [_areaTableView reloadData];
            
            NSIndexPath *lastprovincepath = [NSIndexPath indexPathForRow:_selprovinceIndex inSection:0];
            UITableViewCell *lastlastcitypathcell = [tableView cellForRowAtIndexPath:lastprovincepath];
            lastlastcitypathcell.accessoryType = UITableViewCellAccessoryNone;
            
            NSIndexPath *lastcitypath = [NSIndexPath indexPathForRow:_selcityIndex inSection:0];
            UITableViewCell *lastcitycell = [_subTableView cellForRowAtIndexPath:lastcitypath];
            lastcitycell.accessoryType = UITableViewCellAccessoryNone;
            
            NSIndexPath *lastareapath = [NSIndexPath indexPathForRow:_selAreaIndex inSection:0];
            UITableViewCell *lastacrecell = [_areaTableView cellForRowAtIndexPath:lastareapath];
            lastacrecell.accessoryType = UITableViewCellAccessoryNone;
            
            UITableViewCell *cell = [tableView cellForRowAtIndexPath:indexPath];
            cell.accessoryType = UITableViewCellAccessoryCheckmark;
        }
        else
        {
            UITableViewCell *cell = [tableView cellForRowAtIndexPath:indexPath];
            cell.accessoryType = UITableViewCellAccessoryCheckmark;
        }
        _selprovinceIndex = indexPath.row;
        
        _clickRefresh = YES;
        
        
    }
    else if ([tableView isEqual:_subTableView])
    {
        _city = _cityList[indexPath.row];
        if (_selcityIndex != indexPath.row)//选择的市有变化
        {
            _selAreaIndex = 0;
            //获得区列表
            _arealist = [_cityDic objectForKey:_city];
            _area = _arealist[_selAreaIndex];
            [_areaTableView reloadData];
            NSIndexPath *lastpath = [NSIndexPath indexPathForRow:_selcityIndex inSection:0];
            UITableViewCell *lastcell = [tableView cellForRowAtIndexPath:lastpath];
            lastcell.accessoryType = UITableViewCellAccessoryNone;
            
            NSIndexPath *lastareapath = [NSIndexPath indexPathForRow:_selAreaIndex inSection:0];
            UITableViewCell *lastacrecell = [_areaTableView cellForRowAtIndexPath:lastareapath];
            lastacrecell.accessoryType = UITableViewCellAccessoryNone;
            
            UITableViewCell *cell = [tableView cellForRowAtIndexPath:indexPath];
            cell.accessoryType = UITableViewCellAccessoryCheckmark;
        }
        else
        {
            UITableViewCell *cell = [tableView cellForRowAtIndexPath:indexPath];
            cell.accessoryType = UITableViewCellAccessoryCheckmark;
        }
        _selcityIndex = indexPath.row;

        
    }
    else
    {
        if (_selAreaIndex != indexPath.row) {
            NSIndexPath *lastpath = [NSIndexPath indexPathForRow:_selAreaIndex inSection:0];
            UITableViewCell *lastcell = [tableView cellForRowAtIndexPath:lastpath];
            lastcell.accessoryType = UITableViewCellAccessoryNone;
            UITableViewCell *cell = [tableView cellForRowAtIndexPath:indexPath];
            cell.accessoryType = UITableViewCellAccessoryCheckmark;
        }
        
        _selAreaIndex = indexPath.row;
        _area = _arealist[_selAreaIndex];
    }
    self.title = [NSString stringWithFormat:@"%@ %@ %@",_province,_city,_area];
}

- (void)didReceiveMemoryWarning {
    [super didReceiveMemoryWarning];
    // Dispose of any resources that can be recreated.
}

/*
#pragma mark - Navigation

// In a storyboard-based application, you will often want to do a little preparation before navigation
- (void)prepareForSegue:(UIStoryboardSegue *)segue sender:(id)sender {
    // Get the new view controller using [segue destinationViewController].
    // Pass the selected object to the new view controller.
}
*/

@end
