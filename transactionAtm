import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.format.annotation.DateTimeFormat;
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.validation.BindingResult;
import org.springframework.web.bind.annotation.*;
import org.springframework.web.servlet.mvc.support.RedirectAttributes;

import java.time.LocalDateTime;
import java.time.temporal.ChronoUnit;
import java.util.List;

@Controller
@RequestMapping("/transactions")
public class TransactionController {

    @Autowired
    private TransactionRepository transactionRepository;

    @Autowired
    private AccountRepository accountRepository;

    // Transaction Report (ListView equivalent)
    @GetMapping("/report")
    public String transactionReport(
            @RequestParam(required = false)
            @DateTimeFormat(pattern = "yyyy-MM-dd") LocalDateTime startDate,
            @RequestParam(required = false)
            @DateTimeFormat(pattern = "yyyy-MM-dd") LocalDateTime endDate,
            Model model,
            @AuthenticationPrincipal User user) {

        Account account = user.getAccount();
        List<Transaction> transactions;

        if (startDate != null && endDate != null) {
            transactions = transactionRepository.findByAccountAndTimestampBetween(account, startDate, endDate);
        } else {
            transactions = transactionRepository.findByAccount(account);
        }

        model.addAttribute("account", account);
        model.addAttribute("transactions", transactions);
        return "transactions/transaction_report";
    }

    // Deposit Money
    @PostMapping("/deposit")
    public String depositMoney(@ModelAttribute DepositForm form,
                               BindingResult result,
                               RedirectAttributes redirectAttributes,
                               @AuthenticationPrincipal User user) {

        if (result.hasErrors()) {
            return "transactions/transaction_form";
        }

        Account account = user.getAccount();
        double amount = form.getAmount();

        if (account.getInitialDepositDate() == null) {
            LocalDateTime now = LocalDateTime.now();
            int nextInterestMonth = 12 / account.getAccountType().getInterestCalculationPerYear();
            account.setInitialDepositDate(now);
            account.setInterestStartDate(now.plusMonths(nextInterestMonth));
        }

        account.setBalance(account.getBalance() + amount);
        accountRepository.save(account);

        Transaction transaction = new Transaction(account, amount, TransactionType.DEPOSIT);
        transactionRepository.save(transaction);

        redirectAttributes.addFlashAttribute("success", amount + "$ was deposited successfully");
        return "redirect:/transactions/report";
    }

    // Withdraw Money
    @PostMapping("/withdraw")
    public String withdrawMoney(@ModelAttribute WithdrawForm form,
                                BindingResult result,
                                RedirectAttributes redirectAttributes,
                                @AuthenticationPrincipal User user) {

        if (result.hasErrors()) {
            return "transactions/transaction_form";
        }

        Account account = user.getAccount();
        double amount = form.getAmount();

        account.setBalance(account.getBalance() - amount);
        accountRepository.save(account);

        Transaction transaction = new Transaction(account, amount, TransactionType.WITHDRAWAL);
        transactionRepository.save(transaction);

        redirectAttributes.addFlashAttribute("success", "Successfully withdrawn " + amount + "$");
        return "redirect:/transactions/report";
    }
}
