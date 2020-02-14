Number.prototype.round = function (decimals) {
    if (decimals === null) {
        return this;
    }
    return Number((Math.round(this + "e" + decimals) + "e-" + decimals));
};

sum = function (arr, round_to = null) {
    return arr.reduce((x, y) => x + y, 0).round(round_to);
};

function get_percent_nom(amount, perc, round_to = null) {
    var raw = (1.0 / (100.0 / perc)) * amount;
    return raw.round(round_to)
}


function get_gross(net, vat, round_to = null) {
    var result = {
        'net': net.round(round_to),
        'vat': vat,
        'vat_nominal': get_percent_nom(net, vat, round_to),
    };
    result['gross'] = result['net'] + result['vat_nominal'];
    return result

}

function get_net(gross, vat, round_to = null) {
    var result = {
        'gross': gross.round(round_to),
        'vat': vat,
    };
    result.vat_nominal = (result.gross - result.gross / (1.0 + vat / 100.0)).round(2);
    result.net = result.gross - result.vat_nominal;
    return result

}

class Item {
    static NET = 'net';
    static GROSS = 'gross';

    constructor(unit_price, net_or_gross, vat, quantity = 1) {
        this._net_unit = null;
        this._gross_unit = null;
        if (net_or_gross === Item.NET) {
            this._net_unit = unit_price;
        } else if (net_or_gross === Item.GROSS) {
            this._gross_unit = unit_price;
        } else {
            throw "net_or_gross options are 'net' or 'gross'"
        }
        this.vat = vat;
        this.quantity = quantity;
        this.net_total = null;
        this.gross_total = null;
        this.vat_total = null;
        this.vat_unit = null;

        if (net_or_gross === Item.NET) {
            this.calculate_from_net_unit();
        } else {
            this.calculate_from_gross_unit();
        }
    }

    set net_unit(price) {
        this._net_unit = price;
        this.calculate_from_net_unit();
    }

    set gross_unit(price) {
        this._gross_unit = price;
        this.calculate_from_gross_unit();
    }
    get net_unit() {
        return this._net_unit;
    }

    get gross_unit() {
        return this._gross_unit;
    }

    calculate_from_net_unit() {
        var result = get_gross(this._net_unit, this.vat, 2);
        this._gross_unit = result.gross;
        this.vat_unit = result.vat_nominal;
        this.calculate_totals()

    }

    calculate_from_gross_unit() {
        var result = get_net(this._gross_unit, this.vat, 2);
        this._net_unit = result.net;
        this.vat_unit = result.vat_nominal;
        this.calculate_totals()
    }

    calculate_totals() {
        this.net_total = (this._net_unit * this.quantity).round(2);
        this.gross_total = (this._gross_unit * this.quantity).round(2);
        this.vat_total = this.vat_unit * this.quantity;
    }

    set_net_unit(price) {
        this.net_unit = price;
        this.calculate_from_net_unit();
    }

    set_gross_unit(price) {
        this.gross_unit = price;
        this.calculate_from_gross_unit();
    }
}

class Invoice {
    static NET = 'net';
    static GROSS = 'gross';

    constructor(items) {
        this.items = items;
        this.vat_nominals = {};
        this.net_total = null;
        this.gross_total = null;
        this.vat_total = null;
        this.calculate();
    }

    calculate() {
        var vats = new Set(this.items.map(x => x.vat));

        for (var vat of vats) {
            var items = this.items.filter(x => x.vat === vat);
            this.vat_nominals[vat] = sum(items.map(x => x.vat_total)).round(2);
        }
        this.gross_total = sum(this.items.map(x => x.gross_total)).round(2);
        this.net_total = sum(this.items.map(x => x.net_total)).round(2);
        this.vat_total = (this.gross_total - this.net_total).round(2);
    }

    addItem(item) {
        this.items.push(item);
        this.calculate();
    }

}
